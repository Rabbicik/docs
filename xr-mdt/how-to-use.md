---
title: "How to Use"
description: "Commands and walkthrough of all XR-MDT modules."
---

All the most important commands and features of **XR-MDT** are described below.

## Commands

| Command | Description | Default Key | Permissions |
| :--- | :--- | :--- | :--- |
| `/mdt` | Opens the main tablet panel | `INSERT` | Jobs (Police, EMS, DOJ) |
| `/mdt_dispatch` | Opens the dispatch view | `DELETE` | Jobs (Police, EMS) |
| `/panic_button` | Panic button (sends an alert to all) | `CTRL + L` | Police, EMS |
| `/bk1, /bk2, /bk3` | Call for backup (Code 1, 2, 3) | - | Police |
| `/10-13` | "Officer Down" alert | - | Police |

## How to trigger the tablet? (Item usage)

If you have a `tablet` item, use it in your inventory to open the panel. You can also configure your own keys in the FiveM menu (`Settings > Key Bindings > FiveM`).

## Modules

### 🚔 LSPD (Police)
*   **Search:** You can search for citizens by FirstName, LastName, or SSN.
*   **Citizen Profile:** View notes, convictions, licenses, and vehicles. You can issue new licenses or fines here.
*   **Incidents:** Create reports from events, add offenders, evidence, and mark location.

### 🚑 EMS (Medics)
*   **Patient Records:** Review patient history.
*   **Alerts:** Receive notifications of unconscious people directly on the dashboard.

### ⚖️ DOJ (Court / Prosecutors)
*   Review records and incidents with limited permissions to some police data.

## Settings
All settings (e.g., threat level colors, vehicle prices in LSPD garage) can be found in the `configs/` folder.
*   `configs/config.lspd.lua` - Police settings.
*   `configs/config.ems.lua` - Medic settings.
*   `configs/config.doj.lua` - Court settings.
