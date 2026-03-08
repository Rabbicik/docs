---
title: "Common Issues"
description: "Troubleshooting guide for the most frequent XR-MDT problems."
---

Here you will find solutions to the most common issues related to **XR-MDT**.

## 🔴 Tablet won't open
*   **Console Error (Client):** Check if `fxmanifest.lua` load all resources correctly.
*   **Console Error (Server):** Often caused by missing database connection or errors in the `editable/` files.
*   **NUI Problem:** If you hear tablet sounds but see nothing, check your FiveM cache or use `appdata/local/FiveM/FiveM.app/data/cache`.

## 💾 Database Errors
*   **Unknown column '...' in 'users/players':** Ensure your tables have the appropriate fields. If using a custom framework, you can adjust the queries in the `editable/server/` folder.
*   **Table 'mdt_reports' doesn't exist:** You didn't import the `xr-mdt.sql` file. Import it and restart the server.

## ⚠️ Notification Problem
*   If you don't see notifications, check if your `ox_lib` version is up to date.
*   In `configs/config.main.lua`, change `Config.Notify.Type` to `'qb'` or `'esx'` depending on your framework.

## 🚑 EMS cannot see their module
*   Check if the `ambulance` job is correctly set in your framework.
*   Ensure the `Config.EnableEMS` section in `configs/config.main.lua` is set to `true`.

## 📍 Dispatch not showing blips
*   Dispatch blips are only created for the jobs mentioned in the `jobs` parameter when calling the export.
*   Check if you have the correct permissions in `configs/config.lspd.lua` (e.g., `dispatch_view = true`).

---

If you still have problems, join our Discord server and check the help (support) channel.
