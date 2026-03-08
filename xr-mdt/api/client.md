# API - Client Side

External resources (e.g., phone, radar, robbery scripts) can communicate with **XR-MDT** client-side using the following features.

## Exports

### `createAlert`
Allows for triggering a dispatch alert on the player's screen. Used for dispatching and robbery scripts.

**Important:** The `jobs` parameter is critical for segregating alerts (LSPD / EMS).
*   **Police (LSPD):** Alerts with `jobs = {'police'}` will go to the LSPD module and be blue.
*   **EMS:** Alerts with `jobs = {'ambulance'}` will go to the EMS module and be red.

**Example usage (Police - Blue):**
```lua
exports['xr-mdt']:createAlert({
    code = '10-31',
    title = 'Store Robbery',
    description = 'Alarm registered at 24/7 store',
    coords = GetEntityCoords(PlayerPedId()),
    sound = 'POLICERobbery',
    blip = {
        sprite = 161,
        scale = 1.0,
        color = 3, -- Blip color
        name = 'Robbery'
    },
    jobs = {'police'} -- Visible only for Police
})
```

**Example usage (EMS - Red):**
```lua
exports['xr-mdt']:createAlert({
    code = '10-47',
    title = 'Injured Person',
    description = 'Unconscious person reported.',
    coords = GetEntityCoords(PlayerPedId()),
    sound = 'EMSAlert',
    blip = {
        sprite = 153,
        scale = 1.0,
        color = 1, -- Red
        name = 'Medical Emergency'
    },
    jobs = {'ambulance'} -- Visible for Medics on EMS Dashboard
})
```

---

### `TriggerDispatch`
A wrapper in `editable/client/main.lua` to simplify sending dispatch alerts directly from an external client script (e.g., jeweler script).

| Argument | Type | Description |
| :--- | :--- | :--- |
| `code` | string | 10-code (e.g., "10-90"). |
| `title` | string | Alert title. |
| `description` | string | Event details. |
| `location` | string | Street name or area. |
| `jobs` | table | Job groups (e.g., `{'police'}`). |

**Example usage:**
```lua
-- Jeweler Robbery (Blue)
exports['xr-mdt']:TriggerDispatch(
    '10-90', 
    'Jewelry Store Robbery', 
    'Reported shots and breaking showcases.', 
    'Vangelico Jewelry', 
    {'police'} -- Sends to LSPD module
)
```

### `openTablet`
Opens the tablet for the current player (automatically detects if they should open LSPD, EMS, or DOJ based on their job).

```lua
exports['xr-mdt']:openTablet()
```

### `getCurrentCode`
Returns the current active LSPD code (threat level) object.

```lua
local code = exports['xr-mdt']:getCurrentCode()
print(code.label) -- e.g., "CODE RED"
```

### `updateCode`
Changes the current LSPD code (threat level).

| Argument | Type | Description |
| :--- | :--- | :--- |
| `codeId` | string | The ID of the code from Config (e.g., 'green', 'red'). |

```lua
exports['xr-mdt']:updateCode('red')
```

---

## Client Events

### `xr-mdt:client:openTablet`
Triggers the tablet opening attempt (checks job and permissions).

```lua
TriggerEvent('xr-mdt:client:openTablet')
```

### `xr-mdt:client:onCodeChange`
Triggered whenever the LSPD code (threat level) is changed. Useful for external scripts to change HUD colors or ambient music.

```lua
RegisterNetEvent('xr-mdt:client:onCodeChange', function(codeData)
    print("New threat level: " .. codeData.label)
end)
```

### `xr-mdt:client:playSiren`
Plays a siren/alert sound in the tablet UI.

```lua
TriggerEvent('xr-mdt:client:playSiren')
```
