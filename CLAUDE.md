# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project shape

A static, dependency-free web app for the Shadowdark RPG. Three self-contained HTML files (`index.html`, `player.html`, `gm.html`) at the repo root, each loading **Tailwind CSS** and **PeerJS** from a CDN. No build step, no package manager, no test suite, no lint config. The site is served by GitHub Pages from `main` / root, so any change to the HTML files is the deploy.

To run locally, open `index.html` in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). The PeerJS room features will not work from `file://` in all browsers — prefer a local server when testing player↔GM sync.

## Architecture

### Two clients connected by PeerJS

`gm.html` is the host. On load it calls `new Peer()` (gm.html:353), and the auto-assigned PeerJS ID becomes the room code players type in. `player.html` calls `dmPeer.connect(code)` (player.html:650) to open a WebRTC data channel. There is no server: signalling goes through the public PeerJS cloud, data flows browser-to-browser. The GM page auto-retries `initPeer` every 4s on error (gm.html:377); the player page times out a connect attempt after 10s (player.html:651).

### Message protocol over the data channel

Both sides exchange plain JS objects with a `type` discriminator. Player → GM:

- `join` — full snapshot of the character (name, ancestry, cls, level, abilities, talents, spells, gear, freeCarry, gp/sp/cp, hp, hpMax, ac). Sent immediately on connect by `sendJoin` (player.html:689).
- `hp_sync` — `{hp, hpMax}` whenever the player edits the HP inputs (player.html:766, wired via the `input` listener at player.html:841).
- `sell_rejected` — `{item, reason}` if the player can't afford a shop sale.

GM → player:

- `set_hp` / `set_hp_max` — push new HP values; player applies them and re-saves (player.html:776).
- `treasure` — `{items:[…], gp, sp, cp}`; player appends each item via `addGearRow` and adds currency (player.html:785).
- `sell_item` — `{item, price, currency}`; player checks balance, deducts, and adds to gear, or sends back `sell_rejected` (player.html:806).

When extending the protocol, add the new `type` to both `handleMsg` (gm.html:381) and `handleDmMsg` (player.html:775), and remember the player snapshot lives only in the GM's in-memory `players[pid]` map — there is no resync request, so the player must re-`sendJoin` if its data shape changes.

### Combatant model couples to connected players

The GM `combatants` array (gm.html:458) mixes NPCs (`peerId: null`, persisted) and connected players (`peerId` set, transient). On `join`, a player is auto-pushed as a combatant (gm.html:407). On `close`, it's filtered out (gm.html:367). HP edits on a player-backed combatant card both mutate `players[pid]` and `send` a `set_hp` (gm.html:497) — keep that two-way write in mind when touching combat HP controls. `saveDashboard` deliberately strips `peerId`-bearing combatants before writing to storage (gm.html:833).

### Persistence

Everything is `localStorage`:

- `shadowdark_sheet` — full player sheet, written by `scheduleSave` (player.html:380, 453) with an 800ms debounce on any `input`/`change`.
- `dm_dashboard` — GM notes, NPC combatants, treasures, and shops (gm.html:830).

`getSheetData` / `loadSheetData` (player.html:383, 419) are the canonical serializers. Several non-`id` text inputs are saved positionally by `input_<index>`, so reordering or inserting fields in the HTML will silently shift previously-saved data into the wrong inputs — when restructuring the sheet markup, prefer giving inputs stable `id`s (or saving by `placeholder`) over keeping the index dance.

### Data catalogs are inline

Large data tables are embedded as JS object literals in the HTML files, not external JSON:

- `CHARGEN` in player.html:865 — ancestries, classes (with `hd`, `features`, `levelTalents`), backgrounds, alignments, deities, names, freeGear, and the `spells` table for Wizard/Priest character creation.
- `QUICKSTART` in gm.html:1067 — `monsters`, `weapons`, `armor`, `gear`, `magic` browsed via the GM's catalog modals.

Both are pure data; adding entries is a literal edit. There is no schema validation, but `pickMonster`/`pickItem`/`applyCharGen` read specific fields (`name`, `ac`, `hp`, `atk`, `desc`, `price`, `currency`, etc.) — keep those keys consistent.

## Conventions

- **Vanilla ES5-ish JS only.** No bundler, no transpiler — `var`, `function(){}`, and `Array.from(...).forEach` are the prevailing style throughout both files. Don't introduce ESM imports or syntax requiring a build.
- **Tailwind classes inline; minimal `<style>` blocks** at the top of each file (player.html:12, gm.html top). The Cinzel/Crimson font pairing and `font-cinzel`/`font-crimson` utilities are extended in the inline `tailwind.config` and used heavily for the OSR look.
- **DOM-driven state.** Most "state" lives in the DOM (input values, list rows). The few module-level vars are `dmPeer`/`dmConn`/`dmConnected` (player) and `peer`/`connections`/`players`/`combatants`/`shops`/`tItems` (GM). When persisting, the code reads back out of the DOM via `getSheetData` / `saveDashboard`.
- **Always escape with `esc()`** (gm.html:328) when building innerHTML from player- or GM-supplied strings. `renderCombat`, `renderTreasure`, `renderAllShops`, and `showCharacterSheet` all build markup as strings.
- **HP changes are the hot path for sync.** Any new code that mutates `#hp` or `#hp-max` on the player side must either dispatch an `input` event or call `syncHpToDm()` directly, otherwise the GM will see stale values.

## Git workflow

Development happens on per-task branches (recent examples: `claude/update-gm-menu-colors-cApOY`, `claude/fix-quickstart-gear-sync-AqJlm`). PRs merge into `main`, which auto-deploys via GitHub Pages.
