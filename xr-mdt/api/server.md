# 🖧 API — Server Side

Nearly all backend routing integrations occur exclusively inside the `editable/server/` folder without exposing core escrowed code. This page covers **server exports**, **secure event triggers**, and **admin commands**. For the full `EditTable` reference, see the dedicated [EditTable Reference](editable.md).

---

## 📤 Server Exports

All exports are registered in `editable/server/` files and available to any external script via `exports['xr-mdt']:ExportName(...)`.

### Logging

| Export | Parameters | Description |
| :--- | :--- | :--- |
| `AddLog` | `(source, job, action_type, details, debugType?)` | Records an audit log to `mdt_logs` DB table and forwards to Discord webhook. |

```lua
exports['xr-mdt']:AddLog(
    source,
    "police",
    "Report: Created",
    "Officer created report #105 for citizen John Doe",
    "General" -- Webhook category from Config.Webhooks.Types
)
```

**`AddLog` Parameter Details:**

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `source` | `number` | ✅ | Server ID of the player performing the action. |
| `job` | `string` | ✅ | Job/faction name (e.g. `'police'`, `'ambulance'`). Used for webhook routing. |
| `action_type` | `string` | ✅ | The action title (e.g. `'Fine Issued'`, `'Warrant Created'`). |
| `details` | `string` | ✅ | Full descriptive details of the action. |
| `debugType` | `string` | ❌ | Webhook category key (e.g. `'Money'`, `'Items'`, `'Database'`). |

---

### Weapons

| Export | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `GenerateWeaponSerial` | `()` | `string` | Generates a unique serial number based on `Config.WeaponSerialization`. |
| `GenerateVin` | `()` | `string` | Generates a unique VIN (verified against DB for uniqueness). |

```lua
local serial = exports['xr-mdt']:GenerateWeaponSerial()
-- e.g. "SN-A1B2-C3D4"

local vin = exports['xr-mdt']:GenerateVin()
-- e.g. "A3BF92KL1MNP"
```

---

### Vehicles

| Export | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `AddVehicle` | `(source, targetId, model, garage, financeData?, ownerOverride?)` | `boolean, plate, vin` | Adds a vehicle to the database with auto-generated plate and VIN. |

```lua
local success, plate, vin = exports['xr-mdt']:AddVehicle(source, targetPlayerId, 'adder', 'pillboxgarage', nil, nil)
if success then
    print("Vehicle added. Plate: " .. plate .. " | VIN: " .. vin)
end
```

**Finance Data Structure** (for financed vehicles):

```lua
local financeData = {
    balance       = 50000,  -- Remaining balance
    paymentamount = 5000,   -- Monthly payment amount
    paymentsleft  = 10,     -- Number of payments remaining
    financetime   = 30      -- Total finance term in days
}
```

**`ownerOverride`** can be:
- A citizenid string — assigns to that citizen.
- `'police'` — generates a `LSPD XXX` plate.
- `'ambulance'` — generates an `EMS XXXX` plate.

---

### Sentences & Medical

| Export | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `AddSentence` | `(citizenId, fine, jailTime, reason, authorName?)` | `void` | Records a conviction in `lspd_convictions` and applies effects (fine + jail). |
| `ApplyTreatment` | `(citizenId, amount, treatmentName)` | `void` | Bills a patient and applies treatment costs. |
| `GiveBonus` | `(citizenId, amount, businessName)` | `void` | Gives a monetary bonus to an employee. |

```lua
-- Issue a conviction from an external script
exports['xr-mdt']:AddSentence('QJB123', 5000, 36, 'Armed Robbery', 'Officer Smith')

-- Bill a patient
exports['xr-mdt']:ApplyTreatment('QJB456', 2500, 'Surgery')

-- Give an employee bonus
exports['xr-mdt']:GiveBonus('QJB789', 1000, 'police')
```

> [!WARNING]
> `AddSentence` broadcasts `TriggerClientEvent('xr-mdt:client:citizenUpdated', -1, citizenId)` after recording. This refreshes the profile on all open MDT instances.

---

### Device Toggles

These exports bypass item checks and directly trigger device actions:

| Export | Parameters | Description |
| :--- | :--- | :--- |
| `tablet` | `(source)` | Opens the MDT tablet for the specified player. |
| `bodycam` | `(source)` | Triggers bodycam toggle (respects item check). |
| `gps` | `(source)` | Triggers GPS toggle (respects item check). |
| `tracking_band` | `(source)` | Triggers tracking band toggle (respects item check). |
| `document` | `(source, item)` | Opens a document viewer for the player. |
| `toggleBodycam` | `(source)` | Force-toggles bodycam (bypasses item check). |
| `toggleGPS` | `(source)` | Force-toggles GPS (bypasses item check). |

```lua
-- Force-open tablet for a player
exports['xr-mdt']:tablet(source)

-- Force-toggle bodycam without item requirement
exports['xr-mdt']:toggleBodycam(source)
```

---

### Database Auditor

| Export | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `getDatabaseAuditor` | `()` | `XRAuditor` table | Returns the database auditor object for validation. |

```lua
local auditor = exports['xr-mdt']:getDatabaseAuditor()
local results = auditor.ValidateDatabase()
print("Found: " .. results.found .. "/" .. results.total)
for _, tbl in ipairs(results.missing) do
    print("MISSING: " .. tbl)
end
```

---

## ⚡ Server Events (NetEvents)

### `xr-mdt:server:triggerDispatch`

Force a dispatch notification to all on-duty players in specified jobs. Can be triggered from a client via `TriggerServerEvent` or from another server script.

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `data.code` | `string` | 10-code identifier (e.g. `'10-99'`). |
| `data.title` | `string` | Headline for the dispatch panel. |
| `data.description` | `string` | Detailed context. |
| `data.location` | `string` | Street / area label. |
| `data.jobs` | `table` | Job names that receive the ping. |
| `data.coords` | `table?` | `{x, y, z}` coordinates for blip. |
| `data.blip` | `table?` | Blip config `{sprite, scale, color, name}`. |
| `data.sound` | `string?` | Sound effect name. |

```lua
-- From a client script
TriggerServerEvent('xr-mdt:server:triggerDispatch', {
    code        = '10-99',
    title       = 'Bank Robbery',
    description = 'Armed hostiles hitting the vault.',
    location    = 'Legion Square',
    jobs        = {'police', 'ambulance'}
})

-- From a server script (no source required)
TriggerEvent('xr-mdt:server:triggerDispatch', {
    code  = '10-31',
    title = 'Store Robbery',
    jobs  = {'police'}
})
```

> [!TIP]
> The dispatch is routed through `EditTable.Events.SendDispatch(data)` in `editable/server/main.lua`. Override that function to add custom behavior (e.g., forward to external CAD, Discord webhook).

---

### `xr-mdt:server:triggerNotification`

Send a NUI notification directly to a player from any script:

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `text` | `string` | Notification text. |
| `type` | `string` | `'success'`, `'error'`, or `'info'`. |
| `length` | `number` | Duration in milliseconds. |

```lua
TriggerNetEvent('xr-mdt:server:triggerNotification', "License granted.", 'success', 5000)
```

---

### `xr-mdt:server:requestOpenMDT`

Sent by the client when the player presses the MDT keybind or command. Internally validates the player's job and triggers `xr-mdt:client:open`:

```lua
-- Triggered automatically by the client — do not call directly.
-- To open the tablet from server, use the export instead:
exports['xr-mdt']:tablet(source)
```

---

## 🔧 Admin Commands (Server)

These commands are registered via `Bridge.RegisterCommand()` and respect framework-specific permission systems.

| Command | Args | Permission | Description |
| :--- | :--- | :--- | :--- |
| `/givevehicle` | `id model garage?` | Admin | Adds a vehicle to the target player's database record. |
| `/testvin` | — | Admin | Generates and prints a test VIN. |
| `/setbodycam` | `id` | Admin | Force-toggles bodycam for a player (bypasses item check). |
| `/setgps` | `id` | Admin | Toggles GPS for a player. |

---

## ⚖️ Conviction System (LSPD)

The sentencing system applies fines and jail simultaneously. Override the logic in `editable/server/lspd.lua`:

```lua
-- editable/server/lspd.lua
EditTable.LSPD.Functions.HandleSentence = function(source, citizenId, fine, jailTime, reason, authorName)
    -- 1. Deduct fine from player's bank (online or offline)
    -- 2. Add fine to police society account
    -- 3. Send player to jail (online) or write to jail_sentences (offline)
    return true
end

-- Backward-compatible global alias
function LSPD_HandleSentence(...)
    return EditTable.LSPD.Functions.HandleSentence(...)
end
```

> [!IMPORTANT]
> For the complete editable API including all `EditTable.Events`, `EditTable.Functions`, `EditTable.Queries`, and per-module breakdowns, see the **[EditTable Reference →](editable.md)**
