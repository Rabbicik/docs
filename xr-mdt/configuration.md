---
title: "Configuration"
description: "Detailed explanation of all configuration files and settings."
---

The **XR-MDT** system is fully configurable using the files in the `configs/` folder. Below you'll find a description of the most important settings.

## `configs/config.main.lua`

The main configuration file of the resource.

| Variable | Description |
| :--- | :--- |
| `Config.Framework` | Framework selection: `'qb'`, `'esx'`, `'qbox'` or `'auto'`. |
| `Config.Language` | Language for the UI and system messages (e.g., `'en'`, `'pl'`). |
| `Config.Notify.Type` | Notification system: `'qb'`, `'esx'`, `'ox'`. |
| `Config.EnableLSPD` | Enable Police module. |
| `Config.EnableEMS` | Enable Medic module. |
| `Config.EnableDOJ` | Enable Court module. |

## `configs/config.lspd.lua`

Settings specific to the Police faction.

### Job name
```lua
Config.LSPD.JobName = 'police' -- Required job name to open the LSPD tablet
```

### Grades & Permissions
You can precisely define what each rank (grade) can do in MDT.
```lua
Config.LSPD.Grades = {
    ['0'] = { -- Grade 0 (Cadet)
        permissions = {
            dashboard_view = true,
            profiles_view = true,
            reports_create = true,
            employees_manage = false -- Cannot fire employees
        }
    },
    ['15'] = { -- Grade 15 (Chief of Police)
        permissions = {
            dashboard_view = true,
            profiles_view = true,
            data_delete = true,
            employees_manage = true -- Boss menu / staff management access
        }
    }
}
```

### Threat Codes (Codes)
Customize the status colors and threat levels.
```lua
Config.LSPD.Codes = {
    { id = 'green',  label = 'CODE GREEN',      color = '#22c55e', description = 'Stable situation.' },
    { id = 'red',    label = 'CODE RED',        color = '#ef4444', description = 'Emergency state.' },
}
```

### Documents
Google Docs/Sheets integration.
```lua
Config.LSPD.Documents = {
    { title = 'LSPD Rules', url = 'https://docs.google.com/document/d/DOC_ID/preview' },
    { title = 'Penal Code',  url = 'https://docs.google.com/spreadsheets/d/SHEET_ID/preview' },
}
```

## `configs/config.ems.lua` / `config.doj.lua`

Work similarly to the LSPD config, allowing customization of `JobName` and permissions for Medics and Lawyers.

---

> [!TIP]
> Always restart the script after changing files in the `configs/` folder for changes to take effect.
