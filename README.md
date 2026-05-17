# OSR Companion

A self-contained OSR TTRPG character sheet and GM dashboard designed for use with Shadowdark RPG, playable in the browser with no installation required.

**Live site:** https://volnuttz.github.io/osr-companion/

---

## Pages

| File | Purpose |
|------|---------|
| `index.html` | Landing hub with links to player and GM pages |
| `player.html` | Player character sheet |
| `gm.html` | GM dashboard |

---

## Player Sheet (`player.html`)

- Ability scores (STR, INT, DEX, WIS, CON, CHA) with automatic modifier calculation
- HP / AC tracking
- Talents, Spells, Gear, and Free Carry item lists (gear slot counter respects STR and the Hauler talent)
- Currency tracking (GP / SP / CP)
- Dice roller (d4–d20) with ability score generator
- Auto-saves to browser localStorage

### GM Session panel

A collapsible panel in the sidebar lets players connect to a live GM session:

1. Enter the room code shared by the GM
2. Click **Join** — the sheet connects via PeerJS (WebRTC, no server needed)
3. Once connected, the GM can see the player's name, class, HP, and AC in real time

While connected:
- HP changes on the sheet are synced to the GM automatically
- The GM can set HP remotely (applied instantly to the sheet)
- Treasure sent by the GM is added to the gear list and currency fields
- Items sold by the GM are added to gear and currency is deducted (purchase is rejected if the player doesn't have enough currency)

---

## GM Dashboard (`gm.html`)

Peer-to-peer session management for the GM. No server or account required — PeerJS generates a room code that players enter on their sheet.

### Session

On load, a room code is generated and displayed in the sidebar menu. Share it with players so they can join. The player count updates as players connect and disconnect.

### Battlefield tab

Combat tracker for encounters. Players are automatically added as combatant cards when they connect.

- **Add Combatant** modal — enter Name, Description, HP, AC, and Attacks for any NPC or monster
- Drag-and-drop reordering of combatant cards (desktop and touch)
- HP controls on each card (+1/-1/+5/-5) update the player's sheet live if the combatant is a connected player
- All NPC combatants are saved to localStorage and persist across page refreshes

### Treasures tab

Send loot to players individually:

- Add items with **+ Item** or currency with **+ Currency**
- Each entry has its own player target selector (All Players or a specific player)
- Click **Send** on any entry to deliver it — sent items are removed from the list
- Treasure entries persist in localStorage

### Shops tab

Create multiple shops and sell items directly to players:

1. Click **+ Add Shop** and enter a shop name via modal
2. Add items with name, price, and currency (GP / SP / CP)
3. Select a connected player from the **Sell To** dropdown
4. Click **Sell** next to any item — it is added to the player's gear and currency is deducted
5. If the player doesn't have enough currency, the sale is rejected and the GM is notified

Shops and their inventory persist in localStorage.

### Notes tab

Free-text area for session notes, encounter ideas, NPC names, etc. Saved automatically to localStorage.

---

## Networking

Player–GM connectivity uses **PeerJS** (WebRTC data channels via the public PeerJS cloud signalling server). No backend, no accounts, no installation.

- The GM's room code is a randomly generated PeerJS peer ID
- All data stays between connected browsers — nothing is stored on a server
- If the PeerJS signalling server is temporarily unreachable, the GM page retries automatically every 4 seconds
- Player connection attempts time out after 10 seconds with a clear error message

---

## Hosting

Served via GitHub Pages from the `main` branch (root directory).
To enable: **Settings → Pages → Deploy from branch → main / (root)**.

---

## License

This project is licensed under the [MIT License](LICENSE).

### Shadowdark RPG Third-Party License

OSR Companion is an independent product published under the Shadowdark RPG Third-Party License and is not affiliated with The Arcane Library, LLC. Shadowdark RPG © 2023 The Arcane Library, LLC.

The "Designed for use with Shadowdark RPG" logos are used in accordance with the [Shadowdark RPG Third-Party License Version 1.1](https://www.thearcanelibrary.com/pages/shadowdark).
