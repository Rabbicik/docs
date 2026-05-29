# 🌐 API - Client Side

External scripts (phone systems, radar systems, robbery resources) can integrate with **XR-MDT** via client-side exports and server events.

---

## 📡 Client Exports

### `createAlert`

Spawns an interactive dispatch alert popup visible to specific jobs. This is the primary export for custom alert systems.

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `code` | `string` | ✅ | 10-code (e.g. `'10-31'`). |
| `title` | `string` | ✅ | Headline displayed in the dispatch panel. |
| `description` | `string` | ✅ | Detailed context for responding units. |
| `coords` | `vector3` | ❌ | World coordinates for the blip. |
| `sound` | `string` | ❌ | Sound effect name (e.g. `'POLICERobbery'`). |
| `blip` | `table` | ❌ | Blip config: `{ sprite, scale, color, name }`. |
| `jobs` | `table` | ✅ | Job names that receive the alert (e.g. `{'police'}`). |
| `location` | `string` | ❌ | Street / area label (auto-generated if omitted). |

```lua
exports['xr-mdt']:createAlert({
    code        = '10-31',
    title       = 'Store Robbery',
    description = 'Alarm registered at 24/7 store',
    coords      = GetEntityCoords(PlayerPedId()),
    sound       = 'POLICERobbery',
    blip = {
        sprite = 161,
        scale  = 1.0,
        color  = 3,
        name   = 'Robbery'
    },
    jobs = {'police'}
})
```

> [!WARNING]
> The `jobs` array is a security gate. Only on-duty players whose `job.name` matches an entry in `jobs` will receive the alert. Use `{'police'}` for LSPD and `{'ambulance'}` for EMS.

---

### `TriggerDispatch`

A simplified wrapper that routes to the server dispatch system. Does not require coordinates or blip data.

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `code` | `string` | Standard 10-code (e.g. `'10-90'`). |
| `title` | `string` | Event name. |
| `description` | `string` | Context description. |
| `location` | `string` | Street / area label. |
| `jobs` | `table` | Target job names. |

```lua
exports['xr-mdt']:TriggerDispatch(
    '10-90',
    'Jewelry Store Robbery',
    'Smash & Grab detected in lower chambers.',
    'Vangelico Jewelry',
    {'police'}
)
```

> [!NOTE]
> `TriggerDispatch` triggers `TriggerServerEvent('xr-mdt:server:triggerDispatch', data)` which then calls `EditTable.Events.SendDispatch(data)` — fully customizable in `editable/server/main.lua`.

---

### `AddDispatch`

Directly adds a dispatch entry to the local NUI panel without going through the server. Use for client-local alerts.

```lua
exports['xr-mdt']:AddDispatch({
    code        = '10-65',
    title       = 'Traffic Stop',
    description = 'Vehicle stopped on Route 68',
    location    = 'Route 68',
    jobs        = {'police'},
    time        = GetGameTimer()
})
```

---

### `openTablet`

Opens the MDT tablet for the local player programmatically (bypasses item check):

```lua
exports['xr-mdt']:openTablet()
```

---

### `getCurrentCode` / `updateCode`

Read and write the active threat level / alert code:

```lua
-- Get current threat code
local code = exports['xr-mdt']:getCurrentCode()
print(code.label) -- e.g. "CODE RED"

-- Set a new code
exports['xr-mdt']:updateCode('red') -- options: 'green', 'yellow', 'orange', 'red'
```

---

## ⚡ Client Events (NetEvents)

### `xr-mdt:client:open`

Received when the server authorizes the tablet to open. Contains the full session payload.

```lua
-- Internal event — handled by editable/client/main.lua
RegisterNetEvent('xr-mdt:client:open', function(data)
    -- data.type        = 'police' | 'ems' | 'doj' | 'business'
    -- data.permissions = table of permission flags
end)
```

### `xr-mdt:client:toggleTablet`

Trigger the tablet open/close sequence from an external script (e.g., from a usable item):

```lua
TriggerClientEvent('xr-mdt:client:toggleTablet', source)
```

### `xr-mdt:client:newDispatch`

Sent by the server when a new dispatch arrives. Forwards data to NUI:

```lua
RegisterNetEvent('xr-mdt:client:newDispatch', function(data)
    -- data contains: code, title, description, location, coords, blip, sound
    print("New dispatch: " .. data.title)
end)
```

### `xr-mdt:client:onCodeChange`

Fires when the threat level code changes. Useful for syncing HUD visuals or sirens:

```lua
RegisterNetEvent('xr-mdt:client:onCodeChange', function(codeData)
    print("New threat level: " .. codeData.label)
    -- Trigger HUD color change, deploy sirens, etc.
end)
```

### `xr-mdt:client:playSiren`

Forces auditory replication on the receiving client:

```lua
TriggerEvent('xr-mdt:client:playSiren')
```

### `xr-mdt:client:updateDuty`

Sent to update duty status display in NUI (primarily used by ESX duty system):

```lua
-- state: 1 = on duty, 3 = off duty
TriggerClientEvent('xr-mdt:client:updateDuty', source, 1)
```

### `xr-mdt:client:citizenUpdated`

Broadcast to all clients when a citizen's MDT data changes (e.g., after a new conviction):

```lua
RegisterNetEvent('xr-mdt:client:citizenUpdated', function(citizenId)
    -- Refresh citizen data if currently viewing this profile
end)
```

### `xr-mdt:client:updateFleet`

Broadcast to all clients after a vehicle ownership transfer:

```lua
RegisterNetEvent('xr-mdt:client:updateFleet', function()
    -- Refresh vehicle fleet view
end)
```

---

## 🎮 Admin Commands (Client)

These commands are available when `Config.Debug = true`:

| Command | Description |
| :--- | :--- |
| `/mdt_debug` | Lists all currently registered target zones with their names and coordinates. |
