# 🚔 Adding a New Police Unit

This step-by-step guide walks you through adding a **new police faction** (e.g. SASP, FIB, Rangers) to the XR-MDT system.

---

## 📋 Prerequisites

Before you begin, make sure you have:

- ✅ A **job** defined in your framework (`qb-core/shared/jobs.lua` or ESX `jobs` table)
- ✅ A logo file ready (`.webp` format recommended, `256×256` px)
- ✅ Access to `configs/config.police.lua` and `configs/config.main.lua`

---

## 🔧 Step 1 — Define the Faction in `config.police.lua`

Open `configs/config.police.lua` and add a new entry to the `Config.PoliceJobs` table. The key **must match** the job name from your framework.

```lua
Config.PoliceJobs = {
    ['police'] = { ... }, -- existing LSPD
    ['sheriff'] = { ... }, -- existing LSSD

    -- ========================================================
    -- SASP — San Andreas State Police (NEW UNIT)
    -- ========================================================
    ['sasp'] = {
        -- 1. DISPLAY DATA
        DisplayName   = 'San Andreas State Police',
        ShortName     = 'SASP',
        LogoFile      = 'images/sasp.webp',    -- place file in web/dist/
        Theme         = 'sasp',                -- CSS theme class
        Colors        = {
            primary     = '#7c3aed',           -- main accent color (purple)
            primaryDark = '#5b21b6',           -- darker variant
            bg          = 'rgba(124, 58, 237, 0.1)'  -- element background
        },

        -- 2. COMMAND / KEYBIND
        TabletCommand = 'saspmdt',   -- /saspmdt opens the tablet
        OpenKey       = 'F8',        -- keyboard shortcut

        -- 3. PREFIXES
        CamPrefix     = 'SASP',     -- Bodycam ID: SASP-XXXXXXXXX
        PlatePrefix   = 'SASP',     -- License plate: SASP 123

        -- 4. GPS
        BlipColor     = 27,         -- Blip color on map (FiveM Blip Color ID)

        -- 5. STATUS CODES (THREAT LEVEL)
        Codes = {
            { id = 'green',  label = 'CODE GREEN',  color = '#22c55e', description = 'Stable situation.' },
            { id = 'yellow', label = 'CODE YELLOW', color = '#eab308', description = 'Heightened awareness.' },
            { id = 'red',    label = 'CODE RED',    color = '#ef4444', description = 'Active threat.' },
            { id = 'black',  label = 'CODE BLACK',  color = '#0f172a', description = 'Critical state.' },
        },

        -- 6. DOCUMENT LINKS
        Documents = {
            { title = 'SASP Handbook', url = 'https://docs.google.com/document/d/YOUR-ID' },
            { title = 'SASP Database', url = 'https://docs.google.com/spreadsheets/d/YOUR-ID' },
        },

        -- 7. PERMISSIONS PER GRADE
        Grades = {
            ['0'] = {
                label = 'Trooper',
                permissions = {
                    dashboard_view    = true,
                    profiles_view     = true,
                    profiles_edit     = false,
                    reports_view      = true,
                    reports_create    = true,
                    reports_delete    = false,
                    warrants_view     = true,
                    warrants_create   = false,
                    warrants_delete   = false,
                    vehicles_view     = true,
                    vehicles_edit     = false,
                    convictions_create = false,
                    employees_view    = true,
                    employees_manage  = false,
                    dispatch_view     = true,
                    cameras_view      = false,
                    admin             = false,
                    code_change       = false,
                    custom_fine       = false,
                    custom_sentence   = false,
                    weapons_view      = true,
                    units_view        = true,
                    units_manage      = false,
                }
            },
            ['1'] = {
                label = 'Senior Trooper',
                permissions = {
                    dashboard_view    = true,
                    profiles_view     = true,
                    profiles_edit     = true,
                    reports_view      = true,
                    reports_create    = true,
                    reports_delete    = false,
                    warrants_view     = true,
                    warrants_create   = true,
                    warrants_delete   = false,
                    vehicles_view     = true,
                    vehicles_edit     = true,
                    convictions_create = true,
                    employees_view    = true,
                    employees_manage  = false,
                    dispatch_view     = true,
                    cameras_view      = true,
                    admin             = false,
                    code_change       = false,
                    custom_fine       = true,
                    custom_sentence   = true,
                    weapons_view      = true,
                    units_view        = true,
                    units_manage      = false,
                }
            },
            ['2'] = {
                label = 'Corporal',
                permissions = {
                    dashboard_view = true, profiles_view = true, profiles_edit = true,
                    reports_view = true, reports_create = true, reports_delete = true,
                    warrants_view = true, warrants_create = true, warrants_delete = false,
                    vehicles_view = true, vehicles_edit = true, convictions_create = true,
                    employees_view = true, employees_manage = false, dispatch_view = true,
                    cameras_view = true, admin = false, code_change = true,
                    custom_fine = true, custom_sentence = true, weapons_view = true,
                    units_view = true, units_manage = true,
                }
            },
            ['3'] = {
                label = 'Sergeant',
                permissions = {
                    dashboard_view = true, profiles_view = true, profiles_edit = true,
                    reports_view = true, reports_create = true, reports_delete = true,
                    warrants_view = true, warrants_create = true, warrants_delete = true,
                    vehicles_view = true, vehicles_edit = true, convictions_create = true,
                    employees_view = true, employees_manage = true, dispatch_view = true,
                    cameras_view = true, admin = false, code_change = true,
                    custom_fine = true, custom_sentence = true, weapons_view = true,
                    units_view = true, units_manage = true,
                }
            },
            ['4'] = {
                label = 'Lieutenant',
                permissions = {
                    dashboard_view = true, profiles_view = true, profiles_edit = true,
                    reports_view = true, reports_create = true, reports_delete = true,
                    warrants_view = true, warrants_create = true, warrants_delete = true,
                    vehicles_view = true, vehicles_edit = true, convictions_create = true,
                    employees_view = true, employees_manage = true, dispatch_view = true,
                    cameras_view = true, admin = true, code_change = true,
                    custom_fine = true, custom_sentence = true, weapons_view = true,
                    units_view = true, units_manage = true,
                }
            },
            ['5'] = {
                label = 'Colonel',
                permissions = {
                    dashboard_view = true, profiles_view = true, profiles_create = true,
                    profiles_edit = true, reports_view = true, reports_create = true,
                    reports_delete = true, warrants_view = true, warrants_create = true,
                    warrants_delete = true, vehicles_view = true, vehicles_edit = true,
                    convictions_create = true, employees_view = true, employees_manage = true,
                    dispatch_view = true, cameras_view = true, admin = true,
                    code_change = true, custom_fine = true, custom_sentence = true,
                    weapons_view = true, units_view = true, units_manage = true,
                }
            },
        },

        -- 8. FACTION VEHICLES (MDT GARAGE)
        Vehicles = {
            { model = 'saspstanier',  label = 'SASP Stanier',         price = 50000,  category = 'patrol' },
            { model = 'saspbuffalo', label = 'SASP Buffalo',          price = 65000,  category = 'patrol' },
            { model = 'saspgranger', label = 'SASP Granger',          price = 75000,  category = 'suv'    },
            { model = 'saspmustang', label = 'SASP Dominator',        price = 90000,  category = 'sport'  },
            { model = 'saspmav',     label = 'SASP Helicopter',       price = 0,      category = 'air'    },
        },

        -- 9. VEHICLE CATEGORIES
        VehicleCategories = {
            ['patrol'] = 'Patrol',
            ['sport']  = 'Interceptors',
            ['suv']    = 'SUV / Offroad',
            ['air']    = 'Aircraft',
        },

        -- 10. CUSTOM VEHICLE IMAGES (optional)
        CustomVehicleImages = {
            -- ['saspstanier'] = 'https://link.to/image.png',
        },

        -- 11. PENAL CODE (optional, can copy from LSPD)
        PenalCode = {
            traffic = {
                label = 'Traffic Violations',
                options = {
                    { name = 'speed',      label = 'Speeding',            jail = 0,  fine = 1000 },
                    { name = 'noLicense',  label = 'Driving w/o License', jail = 0,  fine = 500  },
                    { name = 'dui',        label = 'DUI',                 jail = 15, fine = 5000 },
                }
            },
            weapons = {
                label = 'Weapons',
                options = {
                    { name = 'illegalWeapon', label = 'Illegal Weapon',   jail = 15, fine = 5000  },
                    { name = 'armedRobbery',  label = 'Armed Robbery',    jail = 30, fine = 10000 },
                }
            },
        },
    },
}
```

---

## 🔧 Step 2 — Activate the Faction

In the same file (`configs/config.police.lua`), add the new job to `Config.EnablePoliceJobs`:

```lua
Config.EnablePoliceJobs = {
    'police',   -- LSPD
    'pd',       -- PD Alias
    -- 'sheriff', -- LSSD
    -- 'bcso',    -- BCSO
    'sasp',     -- ← NEW UNIT: San Andreas State Police
}
```

> [!IMPORTANT]
> If you don't add the job to `Config.EnablePoliceJobs`, the faction will **not** register any commands or tablet keybinds — even if it's defined in `Config.PoliceJobs`.

---

## 🔧 Step 3 — Logo & Assets

1. Prepare a logo in `.webp` format (recommended `256×256` px)
2. Place the file in:
   ```
   web/dist/images/sasp.webp
   ```
3. The filename must match the `LogoFile` value in your config:
   ```lua
   LogoFile = 'images/sasp.webp'
   ```

---

## 🔧 Step 4 — Bodycam Access (optional)

To allow your SASP officers to use bodycams, add `'sasp'` to the lists in `configs/config.main.lua`:

```lua
Config.Cameras = {
    AllowedJobs = { "police", "pd", "ambulance", "sasp" },  -- ← add here
    ViewerJobs  = { "police", "pd", "ambulance", "sasp" },  -- ← and here

    JobPrefixes = {
        police    = "POLICE",
        ambulance = "AMBULAN",
        sasp      = "SASP",      -- ← new prefix
    },
    -- ...
}
```

---

## 🔧 Step 5 — Helicopter Spawn (optional)

To add a helicopter spawn point for SASP, add an entry in `configs/config.main.lua`:

```lua
Config.HeliSpawns = {
    LSPD = { ... },
    EMS  = { ... },
    SASP = {                                                    -- ← new spawn
        PedCoords    = vector4(x, y, z, heading),               -- NPC position
        PedModel     = 's_m_y_cop_01',                          -- NPC model
        SpawnCoords  = vector4(x, y, z, heading),               -- helicopter spawn
        MinGrade     = 4,                                       -- minimum grade
        VehicleModel = 'polmav',                                -- vehicle model
        Livery       = 2,                                       -- livery ID
        Job          = 'sasp'                                   -- job name
    },
}
```

---

## 🔧 Step 6 — Map Blips (optional)

Add a blip for the SASP headquarters:

```lua
Config.Blips = {
    -- ... existing blips ...
    {
        Coords = vector3(x, y, z),   -- station coordinates
        Sprite = 60,                  -- blip icon
        Color  = 27,                  -- color (should match BlipColor in the faction)
        Scale  = 0.8,
        Label  = 'SASP HQ'
    },
}
```

---

## 🔧 Step 7 — Printer (optional)

```lua
Config.Printers = {
    -- ... existing printers ...
    {
        Coords   = vector3(x, y, z),  -- printer coordinates at SASP base
        Label    = 'SASP Printer',
        Distance = 20.0
    },
}
```

---

## 🔧 Step 8 — Restart & Verify

```
ensure xr-mdt
```

> [!WARNING]
> You **must** fully restart the resource (`ensure xr-mdt`), a simple `refresh` is not enough. Config files are loaded at startup.

---

## ✅ Checklist

| Step | Status |
| :--- | :--- |
| Job defined in framework (qb/esx) | ☐ |
| Entry in `Config.PoliceJobs['sasp']` | ☐ |
| Added to `Config.EnablePoliceJobs` | ☐ |
| Logo `.webp` placed in `web/dist/images/` | ☐ |
| Grades and permissions configured | ☐ |
| Vehicles added (`Vehicles`) | ☐ |
| Penal code configured (`PenalCode`) | ☐ |
| Bodycam jobs updated (optional) | ☐ |
| HeliSpawn added (optional) | ☐ |
| Blips and printers added (optional) | ☐ |
| Resource restarted | ☐ |

---

## 📝 Permissions Reference

| Permission | Description |
| :--- | :--- |
| `dashboard_view` | View the dashboard with statistics |
| `profiles_view` | Browse citizen profiles |
| `profiles_edit` | Edit profiles (notes, photos) |
| `profiles_create` | Create new profiles |
| `reports_view` | Browse reports |
| `reports_create` | Create reports |
| `reports_delete` | Delete reports |
| `warrants_view` | Browse warrants |
| `warrants_create` | Issue warrants |
| `warrants_delete` | Remove warrants |
| `vehicles_view` | Browse the vehicle registry |
| `vehicles_edit` | Edit vehicle data |
| `convictions_create` | Issue sentences (jail + fine) |
| `employees_view` | Browse the employee list |
| `employees_manage` | Manage employees (promote, demote) |
| `dispatch_view` | Access the dispatch panel |
| `cameras_view` | Access the cameras tab |
| `admin` | Full administrative permissions |
| `code_change` | Change the status code (threat level) |
| `custom_fine` | Issue fines with a custom amount |
| `custom_sentence` | Issue sentences with custom jail time |
| `weapons_view` | Browse the weapon registry |
| `units_view` | View units |
| `units_manage` | Manage units |

---

© XR-Core Systems | Professional FiveM Resources
