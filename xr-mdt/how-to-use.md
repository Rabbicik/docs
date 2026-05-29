# How to Use

Discover everything you need to execute rapid workflows seamlessly with **XR-MDT**.

## 🎬 Commands & Keybinds

| Command | Default Key | Description | Access |
| :--- | :--- | :--- | :--- |
| `/mdt` | `DELETE` | Toggles the MDT tablet | All configured jobs |
| `/mdt_dispatch` | — | Toggles the Dispatch overlay panel | LSPD, EMS |
| `/panic_button` | `CTRL + L` | Sends a panic button alert (10-sec cooldown) | LSPD, EMS |
| `/bodycam` | — | Mounts / Unmounts Bodycam | LSPD, EMS |
| `/bk1` | — | Sends a Code 2 backup alert | LSPD |
| `/bk2` or `/bk` | — | Sends a Code 3 backup alert | LSPD |
| `/bk3` | — | Sends a Code 3 critical backup alert | LSPD |
| `/10-13` | — | Triggers an "Officer Down" alert to all emergency services | LSPD |

> [!TIP]
> Key bindings can be overridden in `Settings → Key Bindings → FiveM` within the GTA V pause menu.

---

## 🔄 How the Tablet Opens

1. Player presses the `/mdt` command (or uses the `tablet` item).
2. The client sends `TriggerServerEvent('xr-mdt:server:requestOpenMDT')`.
3. The server validates the player's job, retrieves grade-specific permissions, and sends `TriggerClientEvent('xr-mdt:client:open', source, data)`.
4. The client routes the `open` signal to the correct app (police, ems, doj, or business).
5. The NUI receives the `open` action with `{ type, permissions, factionConfig, locales }`.

---

## 📑 The A4 Document System

**XR-MDT** integrates a beautifully designed, strictly symmetrical multi-page layout engine.

1. **Creating Documents:** Navigate to the `Documents` tab and click `New Document` or select a Template.
2. **Page Pagination:** Standardized A4 dimensions. Text automatically paginating into new pages on overflow.
3. **Drafting and Signatures:** Drop signature slots that use magnetic snapping. Upon signing, real-time sync establishes immutable timestamps.
4. **Physicality:** Printing pushes a `document` item with embedded metadata (`doc_id`, `title`, `html_content`) to the player's inventory.

See the full guide: [📄 A4 Document Engine](features/documents.md)

---

## 📹 WASM-Powered Bodycams

Equip the `bodycam` item or use `/bodycam` to toggle the camera.

* Your camera feed will appear in the `Cameras` tab of the MDT for colleagues to view.
* Video encoding runs in the client's WASM thread — **no server-side processes** (FFmpeg etc.) are needed.
* `IdleFPS` (low framerate buffer) keeps the stream responsive for viewers without impacting server performance.

---

## 📡 GPS & Tracking

* **GPS Tracker:** Equip the `gps` item to appear on the dispatch map for your job.
* **Tracking Band:** Equip the `tracking_band` item to track a citizen on the MDT map (e.g., ankle monitor).
* Offline/disconnected players are automatically removed from the map — no ghost blips.

---

## 🗺️ Dispatch System

Dispatch alerts can be created:
- From within the MDT UI (manual entry).
- Via the `/panic_button` command or `/bk` commands.
- Via external script exports: `exports['xr-mdt']:createAlert(data)` or `exports['xr-mdt']:TriggerDispatch(...)`.
- Via the server event: `TriggerEvent('xr-mdt:server:triggerDispatch', data)`.

---

## 🏬 Specific Job Modules

### 🚔 Police (LSPD)
* Citizen search by name, SSN, or citizen ID.
* Vehicle search by plate or model.
* Issue fines, warrants, and multi-charge sentences (fine + jail simultaneously).
* Grant/revoke licenses (driver, weapon, etc.).
* Manage the weapon armory (kits and stock).
* Track units on the live dispatch map.

### 🚑 EMS
* Access medical records and patient history.
* Bill patients (online: direct bank deduction, offline: phone invoice).
* Revive players from the MDT (triggers `hospital:client:Revive`).
* View emergency dispatch calls with precise coordinates.

### ⚖️ DOJ
* Manage court cases with suspect/vehicle/officer linkage.
* Full citizen and vehicle lookup with warrant and evidence integration.
* Protected from police data wipes — court documents require DOJ-specific authorization.

### 🏢 Business (BizPad)
* Manage employee roster: hire, fire, promote, suspend.
* Update callsigns/badges and add HR notes.
* View financial dashboard: revenue, expenses, weekly charts, recent transactions.
* Integrates with `bank_statements` table for financial history.
