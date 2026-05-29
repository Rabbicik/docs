# Installation Guide

Follow these verified steps for a complete, zero-friction integration of **XR-MDT**, natively engineered for your framework.

## 1. ⚙️ Prerequisites

You must have the following resources running before starting XR-MDT:

| Dependency | Required | Notes |
| :--- | :--- | :--- |
| `oxmysql` | ✅ Yes | All database operations use `MySQL.query.await` / `MySQL.insert.await` etc. |
| `ox_lib` | ✅ Yes | Used for callbacks (`lib.callback`), notifications, and menus. |
| `qb-core` / `es_extended` / `qbx_core` | ✅ One of these | Your selected Framework Core. |

> [!NOTE]
> The `fxmanifest.lua` also uses `@ox_lib/init.lua` as a shared script. Make sure `ox_lib` is started **before** `xr-mdt` in your `server.cfg`.

---

## 2. 📦 Core Setup

1. **Download:** Grab the latest `xr-mdt` release from your Keymaster portal.
2. **Extract:** Unarchive directly into your server's `resources/` folder.
3. **Database Setup:** Import the SQL file from the `INSTALL/` folder into your database. The following tables are required:

**Core & Logs:**
- `mdt_logs` — Central audit log storage
- `mdt_settings` — Global MDT settings

**Police (LSPD) Tables:**
- `lspd_reports` — Police incident reports
- `lspd_citizen_data` — Mugshots and citizen MDT data
- `lspd_convictions` — Criminal conviction records
- `lspd_licenses` — Driver/weapon license tracking
- `lspd_warrants` — Active warrants
- `lspd_incidents` — Incident records

**EMS Tables:**
- `ems_citizen_data` — Patient records
- `ems_medical_records` — Medical history

**DOJ Tables:**
- `doj_cases` — Court case records

**Business Tables:**
- `mdt_business_data` — Business panel data

**Armory Tables:**
- `mdt_armory_kits` — Weapon kit templates
- `mdt_armory_stock` — Current armory stock

> [!IMPORTANT]
> XR-MDT automatically validates all required tables on every resource start and prints missing tables to the server console. If any table is missing, you will see a `^1INCOMPLETE^7` status in the splash screen.

4. **Initial Boot:** Add `ensure xr-mdt` into your `server.cfg`. Ensure it is placed **after** `ox_lib` and your framework core.

---

## 3. 🎯 Integration Guidelines by Framework

### 🔵 QB-Core & Qbox Core

1. Framework is auto-detected. XR-MDT checks for `qbx_core` first, then `qb-core`.
2. Charinfo (firstname, lastname, phone) is read from `PlayerData.charinfo`.
3. The citizen unique identifier used is `PlayerData.citizenid`.

> [!TIP]
> For Qbox (`qbx_core`), the bridge uses `exports.qbx_core:GetPlayer(source)` — this is handled automatically.

### 🟢 ESX Legacy (Latest)

ESX relies on the `users` table. XR-MDT requires two additional columns for full functionality.

#### Required Column Migration
Apply the following SQL if you are using ESX:

```sql
-- Required: SSN for citizen identification
ALTER TABLE `users`
ADD COLUMN IF NOT EXISTS `ssn` VARCHAR(20) DEFAULT NULL;

-- Required: Character ID column (used as citizenid equivalent)
ALTER TABLE `users`
ADD COLUMN IF NOT EXISTS `charid` VARCHAR(50) DEFAULT NULL;
```

> [!IMPORTANT]
> If these columns are missing, XR-MDT will print an error to the console on every player load. Import `INSTALL/database_esx.sql` for a complete schema.

On startup, XR-MDT automatically populates `ssn` for any users who do not have one yet.

#### Vehicle Identification Requirements
For ESX vehicle tracking in the MDT, the `owned_vehicles` table also needs these columns:

```sql
ALTER TABLE `owned_vehicles`
ADD COLUMN IF NOT EXISTS `model` VARCHAR(50) DEFAULT NULL,
ADD COLUMN IF NOT EXISTS `hash` VARCHAR(50) DEFAULT NULL,
ADD COLUMN IF NOT EXISTS `vin` VARCHAR(50) DEFAULT NULL;
```

#### Synchronizing Phone Numbers
If your phone script stores the phone number under a different column name, update this in `config.main.lua`:

```lua
Config.ESXPhoneNumberColumn = 'phone_number' -- Default
```

---

## 4. 📱 Item Schemas

XR-MDT uses the following item names (configurable in `Config.Items` in `config.main.lua`):

| Item Name | Default Key | Purpose |
| :--- | :--- | :--- |
| `tablet` | `Config.Items.tablet` | Opens the MDT |
| `bodycam` | `Config.Items.bodycam` | Toggles bodycam |
| `gps` | `Config.Items.gps` | Toggles GPS tracker |
| `document` | `Config.Items.document` | Views a printed document |
| `tracking_band` | `Config.Items.tracking_band` | Toggles ankle tracking band |

**1. Ox-Inventory (`data/items.lua`)**

```lua
['tablet']   = { label = 'Tablet',           weight = 500, stack = false, close = true },
['bodycam']  = { label = 'Bodycam',          weight = 150, stack = false, close = true },
['gps']      = { label = 'Tracker',          weight = 100, stack = false, close = true },
['document'] = {
    label = 'Printed Document',
    weight = 10,
    stack = false,
    close = true,
    client = {
        export = 'xr-mdt.document'
    }
}
```

> [!TIP]
> For `document`, the `client = { export = 'xr-mdt.document' }` syntax ensures the item opens on the **client side** via the client export. This is the recommended approach for `ox_inventory`.

**2. QB-Core (`shared/items.lua`)**

```lua
['tablet']   = { name='tablet',   label='Tablet',            weight=500,  type='item', unique=false, useable=true, shouldClose=true },
['bodycam']  = { name='bodycam',  label='Bodycam',           weight=150,  type='item', unique=true,  useable=true, shouldClose=true },
['gps']      = { name='gps',      label='Tracker',           weight=100,  type='item', unique=false, useable=true, shouldClose=true },
['document'] = { name='document', label='Printed Document',  weight=10,   type='item', unique=true,  useable=true, shouldClose=true },
```

> [!TIP]
> If `Config.InventoryAutoRegister = true` is set in `config.main.lua`, XR-MDT automatically registers all usable items using `Bridge.RegisterUsableItem`. No extra code needed in your inventory script (except for `tablet` — see below).

---

## 5. 🖥️ Tablet Item Registration

The `tablet` item is **not** auto-registered via the item loop. Instead, it is opened by a **server-side event** triggered by the client command/keybind.

If you want players to open the MDT by **using the tablet item** from their inventory (instead of the `/mdt` command), register it manually in your inventory:

```lua
-- For QB / Qbox (in qb-core/server/main.lua or similar)
QBCore.Functions.CreateUseableItem('tablet', function(source)
    TriggerClientEvent('xr-mdt:client:toggleTablet', source)
end)

-- Or via the export (works from any server script)
exports['xr-mdt']:tablet(source)
```

---

## 6. 💰 Finance & Account Sync

XR-MDT supports most banking resources by resolving them automatically via `Bridge.Bank` in `editable/server/main.lua`.

**Auto-detected banking resources (in order of priority):**

1. `Renewed-Banking` — `exports['Renewed-Banking']:getAccountMoney(accountName)`
2. `xr-bank` — `exports['xr-bank']:GetAccountBalance(accountName)`
3. `qbx_management` — `exports['qbx_management']:GetAccount(accountName).amount`
4. `qb-banking` — `exports['qb-banking']:GetAccountBalance(accountName)`
5. `qb-management` — `TriggerEvent('qb-management:server:GetAccount', ...)`
6. `esx_addonaccount` — Society accounts via `GetSharedAccount(society)`

> [!NOTE]
> All banking transactions (fines, bills, bonuses) are automatically recorded in the `bank_statements` table when using the Bridge methods.

To use a completely custom banking resource, override the `Bridge.Bank` functions in `editable/server/main.lua`:

```lua
function Bridge.Bank.GetBalance(accountName)
    return exports['my-custom-bank']:FetchMoney(accountName)
end

function Bridge.Bank.AddMoney(accountName, amount, reason)
    return exports['my-custom-bank']:AddMoney(accountName, amount)
end

function Bridge.Bank.RemoveMoney(accountName, amount, reason)
    return exports['my-custom-bank']:RemoveMoney(accountName, amount)
end
```