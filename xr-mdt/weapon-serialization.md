# Weapon Serialization & Evidence System

The XR-MDT includes a robust weapon serialization system that allows you to generate unique serial numbers, register weapon ownership, and track them in the evidence database (`mdt_weapons`).

---

## 1. How it Works

Serial numbers are generated using the `GenerateWeaponSerial` export. The generation logic is implemented in `editable/server/main.lua` as `EditTable.LSPD.Functions.GenerateWeaponSerial()`.

The serial format follows: `{Prefix}-{Format}` where `X` in the format string is replaced with a random alphanumeric character (0-9, A-Z).

Generated serial numbers should be stored in the weapon's **item metadata** in your inventory script. When a weapon is checked via the MDT, the system looks up the serial number in the `mdt_weapons` database table.

---

## 2. Configuration

Customize the serial number format in `configs/config.main.lua`:

```lua
Config.WeaponSerialization = {
    Enabled = true,
    Prefix  = 'SN',             -- Serial Number Prefix (e.g., SN-XXXX-XXXX)
    Format  = 'XXXX-XXXX-XXXX', -- X = random alphanumeric character (0-9, A-Z)
}
```

With the default configuration, generated serials look like: `SN-A3B2-K9X1-MNP7`

> [!NOTE]
> The `GenerateWeaponSerial` function does **not** currently check the database for uniqueness (unlike `GenerateVin` which does). The randomness of the format is relied upon for uniqueness. For guaranteed uniqueness, add a DB check inside `EditTable.LSPD.Functions.GenerateWeaponSerial` in `editable/server/main.lua`.

---

## 3. Inventory Integration

### ox_inventory

To generate a serial for a weapon upon receiving it (e.g., in a shop or crafting script):

```lua
-- Example: Giving a weapon with a serial number in ox_inventory
local serial = exports['xr-mdt']:GenerateWeaponSerial()
local metadata = {
    serial      = serial,
    registered  = true  -- Optional flag
}

exports.ox_inventory:AddItem(source, 'weapon_pistol', 1, metadata)

-- Register it in the MDT database
exports['xr-mdt']:RegisterWeapon(citizenid, serial, 'Pistol')
```

### qb-inventory

In QB inventories, serial numbers are typically stored in the `info` table:

```lua
local serial = exports['xr-mdt']:GenerateWeaponSerial()
local info = {
    serie   = serial,
    quality = 100
}

Player.Functions.AddItem('weapon_pistol', 1, false, info)

-- Register it in the MDT database
exports['xr-mdt']:RegisterWeapon(citizenid, serial, 'Pistol')
```

### qs-inventory (Quasar)

```lua
local serial = exports['xr-mdt']:GenerateWeaponSerial()
local info = {
    serie = serial
}
exports['qs-inventory']:AddItem(source, 'weapon_pistol', 1, nil, info)

-- Register it in the MDT database
exports['xr-mdt']:RegisterWeapon(citizenid, serial, 'Pistol')
```

---

## 4. API Reference (Exports)

### `GenerateWeaponSerial`

Generates a unique serial number based on your `Config.WeaponSerialization` config.

**Returns:** `string` — The generated serial (e.g., `"SN-A1B2-C3D4-E5F6"`)

```lua
local serial = exports['xr-mdt']:GenerateWeaponSerial()
```

### `GenerateVin`

Generates a unique Vehicle Identification Number (12-character alphanumeric), **verified against the database** for uniqueness.

**Returns:** `string` — The unique VIN.

```lua
local vin = exports['xr-mdt']:GenerateVin()
```

### `RegisterWeapon`

Registers a weapon in the MDT evidence system (`mdt_weapons` table).

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `citizenid` | `string` | The owner's unique identifier. |
| `serial` | `string` | The unique serial number. |
| `model` | `string` | The weapon model name / label (e.g. `'Combat Pistol'`). |

**Returns:** `boolean` — Success

```lua
exports['xr-mdt']:RegisterWeapon('QJB12345', 'SN-A1B2-C3D4', 'Combat Pistol')
```

### `GetWeaponOwner`

Fetches ownership and history details for a specific serial number from `mdt_weapons`.

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `serial` | `string` | The serial number to look up. |

**Returns:** `table|nil` — Contains: `serial`, `owner` (name), `citizenid`, `model`, `date`

```lua
local info = exports['xr-mdt']:GetWeaponOwner('SN-A1B2-C3D4')
if info then
    print('Owned by: ' .. info.owner)
    print('Registered on: ' .. info.date)
end
```

---

## 5. Script Integration Hook

If you want to automatically register weapons when they are processed by an inventory or weapon shop, hook into your script:

```lua
-- Example: In your weapon shop script
AddEventHandler('weaponshop:weaponSold', function(source, weaponModel, citizenid)
    local serial = exports['xr-mdt']:GenerateWeaponSerial()

    -- Add to inventory with metadata
    exports.ox_inventory:AddItem(source, weaponModel, 1, { serial = serial })

    -- Register in MDT
    exports['xr-mdt']:RegisterWeapon(citizenid, serial, weaponModel)
end)
```

> [!IMPORTANT]
> The MDT only tracks weapons that have been **explicitly registered** via `RegisterWeapon`. It does NOT automatically scan all inventory items for performance reasons. Always call the registration export when a new weapon is issued to a player.

---

## 6. Armory System

In addition to the serialization system, XR-MDT includes a dedicated **Armory** module (`server/sv_armory.lua`, `client/cl_armory.lua`) for law enforcement.

The Armory system tracks:
- **Kits** (`mdt_armory_kits`) — Predefined weapon loadout templates.
- **Stock** (`mdt_armory_stock`) — Current available quantities per kit.

Armory access is permission-gated via `armory_access = true` in the job's Grades config.

---

© XR-Core Systems | Professional FiveM Resources
