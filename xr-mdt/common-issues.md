# Common Issues

Welcome to our troubleshooting baseline. Identify rapid solutions for frequent issues in **XR-MDT**.

---

## 🔴 Tablet / Interface Issues

### Tablet Never Opens
- Check the F8 console for red errors.
- Ensure `ox_lib` is started **before** `xr-mdt` in `server.cfg`.
- Verify `ensure xr-mdt` is present in `server.cfg`.
- Check that the `web/dist/index.html` file exists and the `ui_page` in `fxmanifest.lua` points correctly.

### NUI Renders Black / Blank
- FiveM's Chromium engine does **not** support `backdrop-filter`. If you have modified NUI files, remove any `backdrop-filter` CSS properties.
- Do not use `rgba(0,0,0,0)` as a container background — use `transparent` or a solid color instead.
- Flush FiveM cache: `%localappdata%/FiveM/FiveM.app/data/cache`

### NUI Ghosting (Clicks heard but nothing visible)
- Flush FiveM cache and restart the resource.

### Tablet Opens but Permissions are Wrong
- Confirm the player's `job.name` matches a key in `Config.PoliceJobs`, or matches `'ambulance'` / `'doj'`.
- Check that `Config.EnablePolice = true` (not `Config.EnableLSPD`).
- For ESX: verify `Config.ESXDuty` is set correctly and `GetESXDutyStatus()` is returning the expected value.

---

## 💾 Database Issues

### Error: Unknown Column `ssn` / `charid`
- You are using ESX without running the migration SQL. Execute `INSTALL/database_esx.sql` and restart.
- Alternatively, run manually:
```sql
ALTER TABLE `users` ADD COLUMN IF NOT EXISTS `ssn` VARCHAR(20) DEFAULT NULL;
ALTER TABLE `users` ADD COLUMN IF NOT EXISTS `charid` VARCHAR(50) DEFAULT NULL;
```

### Error: Table `mdt_reports` / `lspd_warrants` / etc. Does Not Exist
- You skipped the SQL import step. Import `INSTALL/xr-mdt.sql` (and `INSTALL/database_esx.sql` for ESX) immediately.
- The startup Database Auditor will list **all missing tables** in the server console. Look for lines beginning with `^1[!] MISSING TABLES DETECTED`.

### Finance Queries Return 0 / Empty
- Finance data (revenue, expenses) is read from the `bank_statements` table.
- This table is only populated when transactions go through `Bridge.Bank.AddMoney()` or `Bridge.Bank.RemoveMoney()`. If your banking resource writes to a different table, override `EditTable.Business.Queries.GetRevenue` in `editable/server/business.lua`.

---

## 📹 Bodycam Issues

### Bodycam Feed Never Appears / Buffers Infinitely
- Lower `Config.Cameras.DefaultFPS` and `Config.Cameras.IdleFPS` in `config.main.lua`.
- Ensure the player's job is in `Config.Cameras.AllowedJobs`.
- The bodycam uses WASM — no server-side FFmpeg. Check the F8 console for NUI errors.
- On servers with 15+ concurrent bodycam users, reduce `DefaultFPS` to avoid NUI thread saturation.

---

## 📑 A4 Document Issues

### Signatures Not Saving / Syncing
- Ensure players are standing still when signing — network tick propagation requires brief stability.
- If scaling appears erratic, check for custom UI scaling mods that break 16:9 NUI constraints.

### Document Item Metadata is Empty
- Ensure the `document` item is registered with `client = { export = 'xr-mdt.document' }` in `ox_inventory`, or a server-side `RegisterUsableItem` in other inventory systems.

---

## 🗺️ GPS / Tracking Issues

### Ghost Blips Persist After Player Disconnects
- The `cl_gps.lua` actively cleans tracking loops on resource stop.
- If blips persist after duty-off, ensure your framework fires an item-drop event when going off duty — the GPS loop requires a server kill trigger.

### Tracking Band Doesn't Appear on MDT Map
- Confirm `Config.Items.tracking_band = 'tracking_band'` matches your inventory item name.
- Ensure `Config.InventoryAutoRegister = true` or manually register the item.

---

## ⚙️ Escrow / Permission Issues

### Cannot Edit Core Logic Files
- Files NOT listed in `escrow_ignore` in `fxmanifest.lua` are Tebex-escrowed. You **cannot** modify them.
- All customizable logic is exposed via `editable/` and the `EditTable` system. See [EditTable Reference](api/editable.md).

### `EditTable.Functions.ErrorCodes` Not Available
- `EditTable` is initialized in `editable/server/main.lua` which loads before all other server scripts. If you see this error, check that `editable/server/main.lua` is listed first in `fxmanifest.lua` server_scripts.

---

## 🐛 Debug Mode

Enable debug mode in `config.main.lua`:
```lua
Config.Debug = true
```

This activates:
- Verbose F8 console logging for all Bridge operations (jobs, money, items, callbacks).
- Visual markers for registered targeting zones in-world.
- The `/mdt_debug` command to list all active target zones.
- Discord debug webhook forwarding for errors.

> [!TIP]
> Always disable `Config.Debug = false` on production servers. Debug mode generates significant log volume and Discord webhook traffic.

---

> [!TIP]
> If all else fails, seek assistance directly via Discord with your exact error code dumps from the F8 console and server console.
