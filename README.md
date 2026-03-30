# OSR Companion

A self-contained OSR TTRPG character sheet and DM dashboard, playable in the browser with no installation required.

**Live site:** https://volnuttz.github.io/osr-companion/

---

## Pages

| File | Purpose |
|------|---------|
| `index.html` | Player character sheet |
| `dm.html` | DM dashboard |

---

## Player Sheet (`index.html`)

- Ability scores (STR, INT, DEX, WIS, CON, CHA) with automatic modifier calculation
- HP / AC tracking
- Talents, Attacks, Spells, Gear, and Free Carry item lists
- Currency tracking (GP / SP / CP)
- Dice roller (d4–d20) with ability score generator
- Auto-saves to browser localStorage

### DM Session panel

A collapsible panel at the top of the sheet lets players connect to a live DM session:

1. Enter the room code shared by the DM
2. Click **Join** — the sheet connects via PeerJS (WebRTC, no server needed)
3. Once connected, the DM can see the player's name, class, HP, and AC in real time

While connected:
- HP changes on the sheet are synced to the DM automatically
- The DM can set HP remotely (applied instantly to the sheet)
- Treasure sent by the DM is added to the gear list and currency fields
- Items sold by the DM are added to gear and gold is deducted

---

## DM Dashboard (`dm.html`)

Peer-to-peer session management for the DM. No server or account required — PeerJS generates a room code that players enter on their sheet.

### Session

On load, a room code is generated and displayed at the top. Share it with players so they can join. The player count updates as players connect and disconnect.

### Players tab

Live view of every connected player:

- Name, class, AC
- HP bar (colour-coded green/yellow/red)
- Editable HP / max HP fields
- Quick adjust buttons: **-5 / -1 / +1 / +5**

Changes made here are sent to the player's sheet in real time.

### Combat tab

Initiative tracker for encounters:

- **Add Combatant** form — enter Name, Description, HP, AC, Initiative, and Attacks for any NPC or monster
- **Sync Connected Players** — pulls all currently connected players into the tracker
- Initiative order is kept sorted automatically; re-entering a value re-sorts immediately
- **Prev / Next** buttons advance the active turn (tracked by combatant ID, not list position, so re-sorting never breaks it)
- HP controls on each card (+1/-1/+5/-5) update the player's sheet live if the combatant is a connected player
- **Clear** removes all combatants after confirmation

### Treasure tab

Send a loot drop to players:

1. Add item names with **+ Add Item**
2. Set GP / SP / CP amounts
3. Choose **All Players** or a specific player from the dropdown
4. Click **Send Treasure** — items appear in the player's gear list and gold is added

### Shop tab

Sell individual items directly to a specific player:

1. Enter a shop name
2. Add items with name, price, and currency (GP / SP / CP)
3. Select a connected player from the **Sell To** dropdown
4. Click **Sell** next to any item — it is added to the player's gear and gold is deducted automatically

### Notes tab

Free-text area for session notes, encounter ideas, NPC names, etc. Saved automatically to localStorage.

---

## Networking

Player–DM connectivity uses **PeerJS** (WebRTC data channels via the public PeerJS cloud signalling server). No backend, no accounts, no installation.

- The DM's room code is a randomly generated PeerJS peer ID
- All data stays between connected browsers — nothing is stored on a server
- If the PeerJS signalling server is temporarily unreachable, the DM page retries automatically every 4 seconds
- Player connection attempts time out after 10 seconds with a clear error message

---

## Hosting

Served via GitHub Pages from the `main` branch (root directory).
To enable: **Settings → Pages → Deploy from branch → main / (root)**.
