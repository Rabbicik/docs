# Installation Guide

Follow these steps for a complete, step-by-step installation of **XR-MDT** tailored for your framework.

## 1. Prerequisites
Ensure you have the following resources installed and started before **XR-MDT**:
*   `oxmysql`
*   `ox_lib`
*   Your choice of framework (`qb-core`, `es_extended`, or `qbox-core`)

---

## 2. General Installation Steps
1.  **Download:** Get `xr-mdt` from Keymaster.
2.  **Extract:** Unzip into your `resources` folder.
3.  **Database:** Import `xr-mdt.sql` into your database.
4.  **Configuration:** Open `configs/config.main.lua` and verify your framework and notification settings.
5.  **Start script:** Add `ensure xr-mdt` to your `server.cfg`.

---

## 3. Framework Specific Setup

### 🔵 QB-Core / Qbox
1.  Open `qb-core/shared/items.lua` and add the tablet item (see section 4).
2.  The script automatically detects these frameworks.
3.  Ensure your `players` table has a `charinfo` column (default in QB).

### 🟢 ESX (Legacy/Latest)
1.  **Database Bridge:** ESX requires specific columns in the `users` table for some MDT features.
2.  **Vehicle System:** For LSPD/EMS vehicle shop integration, you must ensure your `owned_vehicles` table is prepared.
3.  **Phone Number Integration:** By default, the script looks for `phoneNumber` in your player data. If your phone script uses a different column or key, adjust it in `configs/config.main.lua`:
    ```lua
    Config.ESXPhoneNumberColumn = 'phone_number' -- Change to your specific column name
    ```

#### 🚘 ESX Vehicle Database Modification
Run the following SQL to ensure your `owned_vehicles` table supports the MDT's vehicle tracking:
```sql
ALTER TABLE `owned_vehicles` 
ADD COLUMN IF NOT EXISTS `model` VARCHAR(50) DEFAULT NULL,
ADD COLUMN IF NOT EXISTS `hash` VARCHAR(50) DEFAULT NULL,
ADD COLUMN IF NOT EXISTS `vin` VARCHAR(50) DEFAULT NULL;
```

#### 🛠 Vehicle Insertion Code Example
If you are modifying a vehicle shop or garage to work with MDT, use the following logic to insert vehicles into the database.

**ESX Example:**
```lua
local xPlayer = ESX.GetPlayerFromId(src)
local vehicleModel = data.model -- e.g. 'pdstanier'
local plate = "LSPD" .. math.random(100, 999)

MySQL.insert.await('INSERT INTO `owned_vehicles` (`owner`, `vehicle`, `plate`, `model`, `hash`, `vin`, `type`, `stored`) VALUES (?, ?, ?, ?, ?, ?, ?, ?)', {
    xPlayer.identifier, 
    json.encode(data.props), 
    plate, 
    vehicleModel, 
    GetHashKey(vehicleModel), 
    exports['xr-mdt']:GenerateUniqueVin(), 
    'car', 
    1
})
```

**QB-Core / Qbox Example:**
```lua
local xPlayer = QBCore.Functions.GetPlayer(src)
local vehicleModel = data.model
local plate = "LSPD" .. math.random(100, 999)

MySQL.insert.await('INSERT INTO `player_vehicles` (`license`, `citizenid`, `vehicle`, `hash`, `mods`, `plate`, `vin`, `garage`, `state`) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)', {
    xPlayer.PlayerData.license, 
    xPlayer.PlayerData.citizenid, 
    vehicleModel, 
    GetHashKey(vehicleModel), 
    json.encode(data.props), 
    plate, 
    exports['xr-mdt']:GenerateUniqueVin(), 
    'pillboxgarage', 
    0
})
```

---

## 4. � Tablet Item Installation

To enable opening the tablet via an inventory item:

### Item Definitions
**QB-Core (`shared/items.lua`):**
```lua
['tablet'] = {['name'] = 'tablet', ['label'] = 'Police Tablet', ['weight'] = 500, ['type'] = 'item', ['image'] = 'tablet.png', ['unique'] = false, ['useable'] = true, ['shouldClose'] = true, ['combinable'] = nil, ['description'] = 'A police tablet for managing records'},
```

**Ox Inventory (`data/items.lua`):**
```lua
['tablet'] = {
    label = 'Police Tablet',
    weight = 500,
    stack = false,
    close = true,
    description = 'A police tablet for managing records',
},
```

### Script Registration
Add this code to `editable/server/main.lua` to make the item usable:
```lua
Bridge.RegisterUsableItem('tablet', function(source)
    TriggerClientEvent('xr-mdt:client:openTablet', source)
end)
```

---

## 5. Post-Installation
1.  Join the server.
2.  Set your job to `police` (or another supported job).
3.  Use the `tablet` item, type `/mdt`, or press `INSERT` to verify it opens.

---

## 6. Banking & Accounts Integration

**XR-MDT** supports most popular banking systems out of the box. The integration is handled through a bridge in `editable/server/main.lua`.

### Supported Systems (Auto-detected):
*   `Renewed-Banking`
*   `xr-bank`
*   `qb-banking` (QB-Core default)
*   `qbx_management` (Qbox default)
*   `esx_addonaccount` (ESX society accounts)

### How to verify or change your banking bridge:
If your server uses a custom banking script or different exports, you can manually adjust the logic in `editable/server/main.lua` under the `Bridge.Bank` section (approx. line 226).

**Functions you can modify:**
*   `Bridge.Bank.GetBalance(accountName)` - Returns the current balance.
*   `Bridge.Bank.AddMoney(accountName, amount)` - Adds money to the account.
*   `Bridge.Bank.RemoveMoney(accountName, amount)` - Removes money from the account.

**Example: Custom Banking Export**
If you use a script like `my-custom-bank`, modify the function like this:
```lua
function Bridge.Bank.GetBalance(accountName)
    -- Your custom logic
    return exports['my-custom-bank']:GetBalance(accountName)
end
```