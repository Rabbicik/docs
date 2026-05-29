# Configuration

The **XR-MDT** system operates via multiple modular configuration files found in the `configs/` folder. This allows precise tuning for performance, permissions, and layout adjustments without touching core code.

---

## 📁 Configuration Files Overview

| File | Purpose |
| :--- | :--- |
| `configs/config.main.lua` | Core settings: framework, language, tablet command, item names |
| `configs/config.police.lua` | Police job definitions, grades, permissions, penal code |
| `configs/config.ems.lua` | EMS job settings and medical permissions |
| `configs/config.doj.lua` | DOJ/Court job settings and grades |
| `configs/config.business.lua` | Business panel job configuration |
| `configs/config.armory.lua` | Armory system settings and kit definitions |
| `configs/config.webhook.lua` | Discord webhook URLs and embed styling |
| `configs/locales/*.lua` | Language files (e.g., `en.lua`, `pl.lua`) |

---

## 🎛️ Main Settings (`config.main.lua`)

The fundamental structure of the script is driven by `config.main.lua`.

### Framework & Language

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `Config.Framework` | `string` | Force a framework: `'qb'`, `'esx'`, `'qbox'`, or `'auto'` for autodetect. |
| `Config.Language` | `string` | Locale file to use: `'en'`, `'pl'`, etc. Must match a file in `configs/locales/`. |
| `Config.Debug` | `boolean` | Enables verbose console logging and Discord debug webhooks. |
| `Config.ESXPhoneNumberColumn` | `string` | Column name in `users` table for phone number (ESX only). Default: `'phone_number'`. |
| `Config.ESXDuty` | `boolean` | If `true`, tracks ESX duty state separately (required for ESX if you use duty system). |

### Tablet Access

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `Config.Tablet.Command` | `string` | Chat command to open the MDT. Default: `'mdt'`. |
| `Config.Tablet.Key` | `string` | Default key binding. Default: `'DELETE'`. |
| `Config.InventoryAutoRegister` | `boolean` | Auto-register usable items for inventory scripts via Bridge. |
| `Config.FivemanageToken` | `string` | FiveManage API token for media uploads (mugshots, avatars). |

### Item Names

```lua
Config.Items = {
    tablet        = 'tablet',
    bodycam       = 'bodycam',
    gps           = 'gps',
    document      = 'document',
    tracking_band = 'tracking_band'
}
```

### Module Toggles

Enable or disable each sub-system independently:

```lua
Config.EnablePolice   = true  -- NOTE: Internal key is 'EnablePolice', not 'EnableLSPD'
Config.EnableEMS      = true
Config.EnableDOJ      = true
Config.EnableBusiness = true
Config.EnableDispatch = true
```

> [!WARNING]
> In `sv_main.lua` the splash screen reads `Config.EnablePolice` (not `Config.EnableLSPD`). Use `EnablePolice` in `config.main.lua`.

### Notification System

```lua
Config.Notify = {
    Type = 'ox'  -- Options: 'ox', 'qb', 'esx'
}
```

---

## 🚔 Job-Specific Settings (`config.police.lua`)

Every police department is defined in `Config.PoliceJobs` using the job name as the key.

### Multi-Faction Architecture

Starting with v2.0, **XR-MDT** supports multiple police departments within a single resource:

```lua
Config.PoliceJobs = {
    ['police'] = {
        DisplayName = 'Los Santos Police Department',
        Theme = 'lspd',
        Colors = {
            primary = '#38bdf8',
            bg = 'rgba(56, 189, 248, 0.1)'
        },
        Grades = {
            ['0'] = {
                permissions = {
                    dashboard_view   = true,
                    profiles_view    = true,
                    reports_create   = true,
                    employees_manage = false
                }
            },
            ['10'] = {
                permissions = {
                    dashboard_view   = true,
                    data_delete      = true,
                    employees_manage = true
                }
            }
        }
    },
    ['sheriff'] = {
        DisplayName = 'Los Santos County Sheriff',
        Theme = 'lssd',
        Colors = {
            primary = '#f59e0b',
            bg = 'rgba(245, 158, 11, 0.1)'
        },
        Grades = { ... }
    }
}
```

> [!TIP]
> Each faction automatically inherits a unique UI color scheme based on the `Colors` table. The `Theme` CSS class is applied to the NUI for visual branding.

### Permission Keys Reference

| Key | Description |
| :--- | :--- |
| `dashboard_view` | Access to the main dashboard |
| `profiles_view` | View citizen profiles |
| `profiles_edit` | Edit citizen profiles (mugshots, notes) |
| `reports_view` | View police reports |
| `reports_create` | Create new reports |
| `reports_edit` | Edit existing reports |
| `data_delete` | Delete records from the system |
| `employees_manage` | Hire, fire, promote employees |
| `warrants_manage` | Issue and delete warrants |
| `licenses_manage` | Grant and revoke licenses |
| `armory_access` | Access the weapon armory |
| `logs_view` | View audit logs |

---

## 📹 Bodycam System Settings

The WASM-powered Bodycam engine processes video encoding directly on the client — no server-side FFmpeg required.

```lua
Config.Cameras = {
    Resolution  = { width = 1280, height = 720 },
    DefaultFPS  = 12,    -- Active rendering framerate
    IdleFPS     = 0.5,   -- Background buffering framerate
    AllowedJobs = { "police", "pd", "ambulance" }
}
```

> [!WARNING]
> Setting `DefaultFPS` too high may cause framerate degradation on low-end client PCs. The encoding is pushed directly to the NUI rendering thread.

---

## 🖨️ Hardware & Printers

Physical printer locations are used by the A4 Document Engine. Players must walk to a printer to claim printed documents.

```lua
Config.Printers = {
    {
        Coords   = vector3(447.88, -973.86, 30.68),
        Label    = 'LSPD Network Printer',
        Distance = 5.0
    },
    {
        Coords   = vector3(296.31, -592.43, 43.29),
        Label    = 'Sheriff Office Printer',
        Distance = 5.0
    }
}
```

---

## 🔫 Weapon Serialization

See [Weapon Serialization](weapon-serialization.md) for complete details.

```lua
Config.WeaponSerialization = {
    Enabled = true,
    Prefix  = 'SN',
    Format  = 'XXXX-XXXX-XXXX'
}
```

---

## 📣 Webhook Configuration (`config.webhook.lua`)

Webhooks are routed by priority: **Specific Type → Job Category → Main Webhook**.

```lua
Config.Webhooks = {
    Main = 'https://discord.com/api/webhooks/...',

    Types = {
        Money    = '',    -- Financial actions
        Items    = '',    -- Inventory operations
        Database = '',    -- SQL operations (debug only)
        JobChange = '',   -- Job/duty changes
    },

    Jobs = {
        POLICE   = 'https://discord.com/api/webhooks/...',
        EMS      = '',
        DOJ      = '',
        BUSINESS = '',
        System   = '',
    },

    Categories = {
        POLICE   = { Title = '🚔 LSPD',    Color = 3447003  },
        EMS      = { Title = '🚑 EMS',     Color = 3066993  },
        DOJ      = { Title = '⚖️ DOJ',     Color = 15844367 },
        BUSINESS = { Title = '🏢 Business', Color = 15105570 },
        System   = { Title = '⚙️ System',  Color = 9807270  },
    }
}
```

> [!IMPORTANT]
> Save and restart the entire resource when updating `configs/*.lua` to re-bind job permissions and locale data correctly.
