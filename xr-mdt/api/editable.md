# 🧩 EditTable — Complete Reference

The `EditTable` is the **heart of XR-MDT customization**. It is a globally accessible Lua table that exposes every integration point the script offers — from money handling and database queries to dispatch routing and job-specific logic. Because XR-MDT is Tebex Escrow-protected, you **cannot** modify core files — but you **can** modify everything inside the `editable/` folder, and the `EditTable` is the bridge that connects your custom code to the protected core.

---

## 📖 What is the EditTable?

| Concept | Explanation |
| :--- | :--- |
| **Purpose** | A global Lua table (`EditTable`) that holds all editable functions, events, and queries the MDT core calls at runtime. |
| **Location** | Defined across files in `editable/server/` and `editable/client/`. |
| **How it works** | The core (escrowed) code calls `EditTable.Events.RemoveMoney(...)` instead of directly calling a framework function. You control _what happens_ inside that function. |
| **Safety** | Modifying `EditTable` functions never breaks escrow — the core only reads from it. |

> [!IMPORTANT]
> Every `EditTable` function listed below is fully editable. Override any of them to adapt XR-MDT to your server's custom scripts, frameworks, or databases.

---

## 📁 File Structure

```
editable/
├── server/
│   ├── main.lua          ← Core bridge, Bridge functions, framework detection, EditTable.Events
│   ├── lspd.lua          ← Police-specific events, queries, helpers
│   ├── ems.lua           ← EMS-specific events and queries
│   ├── doj.lua           ← DOJ/Court events and queries
│   ├── business.lua      ← Business management events, queries, finance
│   ├── sv_items.lua      ← Item registration & tablet access controller (OpenTablet, exports)
│   └── sv_vehicles.lua   ← Vehicle database operations & VIN generation (AddVehicle export)
└── client/
    ├── main.lua          ← Client bridge, animations, dispatch, UI locales, targeting
    ├── lspd.lua          ← Police client commands (panic, backup, dispatch keybinds)
    ├── ems.lua           ← EMS client hooks (stub — add your logic)
    ├── doj.lua           ← DOJ client hooks (stub — add your logic)
    └── business.lua      ← Business client hooks (stub — add your logic)
```

> [!NOTE]
> Load order matters. `editable/server/main.lua` is loaded **first** among all editable files, so the `Bridge` and `EditTable` tables are available when `lspd.lua`, `ems.lua` etc. are loaded.

---

## 🔧 Server — Core Module (`editable/server/main.lua`)

This is the **main bridge** between XR-MDT and your framework. It contains:
- Framework detection and object initialization (`QBCore`, `ESX`).
- The `Bridge` abstraction layer for all framework operations.
- `EditTable.Events` — the primary integration points called by the core.
- `EditTable.Functions` — utility helpers.
- Event listeners for dispatch and notifications.

---

### Bridge Functions

The `Bridge` table provides a framework-agnostic API. These functions are used internally by `EditTable` and by `editable/server/lspd.lua`, `ems.lua`, etc.

#### Player & Job

| Function | Signature | Description |
| :--- | :--- | :--- |
| `Bridge.GetPlayer` | `(source) → player` | Returns the raw framework player object. |
| `Bridge.GetJob` | `(source) → job` | Returns the player's current job table (normalized for ESX). |
| `Bridge.GetCitizenId` | `(player) → string` | Returns the player's unique identifier (`citizenid` / `identifier`). |
| `Bridge.GetCitizenIdFromObject` | `(player) → string` | Same, but from the player object directly. |
| `Bridge.GetCharInfo` | `(player) → table` | Returns `{ firstname, lastname, phone }`. |
| `Bridge.GetCitizenName` | `(citizenId) → string` | DB lookup — returns `"Firstname Lastname"` for a citizenId. |
| `Bridge.GetSource` | `(player) → number` | Returns the server source ID from the player object. |
| `Bridge.GetMetaData` | `(player, key) → any` | Gets a metadata value from the player. |
| `Bridge.SetMetaData` | `(player, key, value)` | Sets a metadata value on the player. |
| `Bridge.IsOnDuty` | `(player) → boolean` | Returns true if the player is on duty. |
| `Bridge.GetPlayerByCitizenId` | `(citizenId) → player?` | Finds an online player by their citizenId. Returns nil if offline. |
| `Bridge.GetPlayers` | `() → table` | Returns all online player objects. |
| `Bridge.GetJobFromObject` | `(player) → job` | Returns the job table directly from the player object. |

#### Money

| Function | Signature | Description |
| :--- | :--- | :--- |
| `Bridge.GetMoney` | `(source, type) → number` | Gets the player's account balance. `type`: `'cash'` or `'bank'`. |
| `Bridge.AddMoney` | `(source, type, amount) → boolean` | Adds money to the player's account. |
| `Bridge.RemoveMoney` | `(source, type, amount) → boolean` | Removes money from the player's account. Returns false if insufficient. |
| `Bridge.DeductMoneyByCitizenId` | `(citizenId, type, amount, reason) → boolean` | Deducts money by citizenId (handles both online and offline players). |

#### Banking (Society)

| Function | Signature | Description |
| :--- | :--- | :--- |
| `Bridge.Bank.GetBalance` | `(accountName) → number` | Gets a society/business account balance. |
| `Bridge.Bank.AddMoney` | `(accountName, amount, reason?) → boolean` | Deposits into a society account. Also records in `bank_statements`. |
| `Bridge.Bank.RemoveMoney` | `(accountName, amount, reason?) → boolean` | Withdraws from a society account. Also records in `bank_statements`. |

#### Items & Inventory

| Function | Signature | Description |
| :--- | :--- | :--- |
| `Bridge.GetItem` | `(source, item) → table?` | Returns item data from the player's inventory. |
| `Bridge.GetItemCount` | `(source, item) → number` | Returns the quantity of an item the player has. |
| `Bridge.AddItem` | `(source, item, count, slot?, metadata?) → boolean` | Adds an item to the player's inventory. |
| `Bridge.RemoveItem` | `(source, item, count, slot?) → boolean` | Removes an item from the player's inventory. |
| `Bridge.RegisterUsableItem` | `(name, handler)` | Registers a usable item. Uses `ox_inventory:registerHook` when available. |

#### Job Management

| Function | Signature | Description |
| :--- | :--- | :--- |
| `Bridge.SetJob` | `(player, job, grade)` | Changes the player's job. |
| `Bridge.SetJobDuty` | `(player, state)` | Sets the player's duty state (on/off). |
| `Bridge.Save` | `(player)` | Forces a player data save (QB). |
| `Bridge.JailPlayer` | `(source, time)` | Sends a player to jail via the detected jail resource. |
| `Bridge.UpdateVehicleOwner` | `(plate, targetCitizenId) → number` | Transfers vehicle ownership in the DB. |

#### UI & Interaction

| Function | Signature | Description |
| :--- | :--- | :--- |
| `Bridge.Notify` | `(source, text, type, length)` | Sends a notification to a player. |
| `Bridge.RegisterCommand` | `(name, help, arguments, restricted, handler)` | Registers a command (QB uses `QBCore.Commands.Add`, ESX uses `ESX.RegisterCommand`). |
| `Bridge.CreateCallback` | `(name, handler)` | Registers a server callback (QB uses `lib.callback.register`, ESX uses `ESX.RegisterServerCallback`). |
| `Bridge.OnPlayerLoaded` | `(handler)` | Registers a handler for player load events. |

---

### EditTable.Events

These are the **primary integration points**. The MDT core calls these functions whenever it needs to perform an action that depends on your server's specific setup.

---

#### `EditTable.Events.RemoveMoney`

Called when the MDT needs to deduct money from a player (fines, taxes, purchases).

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `source` | `number` | Server ID of the player. |
| `type` | `string` | Account type: `'cash'` or `'bank'`. |
| `amount` | `number` | Amount to deduct. |

**Returns:** `boolean` — `true` if successful.

```lua
EditTable.Events.RemoveMoney = function(source, type, amount)
    return Bridge.RemoveMoney(source, type, amount)
end
```

---

#### `EditTable.Events.HasItem`

Called to check if a player possesses a specific item.

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `source` | `number` | Server ID of the player. |
| `item` | `string` | Item name (e.g. `'id_card'`). |
| `count` | `number?` | Minimum quantity required (defaults to `1`). |

**Returns:** `boolean`

```lua
EditTable.Events.HasItem = function(source, item, count)
    local owned = Bridge.GetItemCount(source, item)
    return owned >= (count or 1)
end
```

---

#### `EditTable.Events.Query`

Every database query in the MDT passes through this function. Override it to use a different SQL library or add query logging.

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `query` | `string` | Raw SQL query string. |
| `params` | `table` | Parameterized values. |

**Returns:** `table` — Query result rows, or `{}` on error.

```lua
EditTable.Events.Query = function(query, params)
    local status, result = pcall(function()
        return MySQL.query.await(query, params)
    end)
    if not status then
        EditTable.Functions.ReportError(EditTable.Functions.ErrorCodes.DATABASE_ERROR,
            string.format("Query Failed: %s\nError: %s", query, result))
        return {}
    end
    return result
end
```

> [!WARNING]
> If you aren't using `oxmysql`, you **must** override this function. The entire MDT depends on it for all database operations.

---

#### `EditTable.Events.SendDispatch`

Controls how dispatch alerts are routed to players.

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `data.code` | `string` | 10-code (e.g. `'10-31'`). |
| `data.title` | `string` | Alert headline. |
| `data.description` | `string` | Detailed description. |
| `data.location` | `string` | Street / area name. |
| `data.jobs` | `table` | Array of job names to target. |
| `data.coords` | `table?` | `{x, y, z}` coordinates. |
| `data.sound` | `string?` | Sound effect name. |
| `data.blip` | `table?` | Blip configuration `{sprite, scale, color, name}`. |

```lua
EditTable.Events.SendDispatch = function(data)
    local jobs = data.jobs or { 'police' }
    local players = GetPlayers()

    for _, src in pairs(players) do
        src = tonumber(src)
        local playerJob = Bridge.GetJob(src)

        if playerJob and playerJob.onduty then
            local jobName = playerJob.name
            for _, targetJob in pairs(jobs) do
                if jobName == targetJob then
                    TriggerClientEvent('xr-mdt:client:newDispatch', src, data)
                    break
                end
            end
        end
    end
end
```

> [!TIP]
> Add your webhook logic or external CAD forwarding inside this function after the player loop.

---

### EditTable.Functions (Server)

| Function | Signature | Description |
| :--- | :--- | :--- |
| `GetPlayer` | `(source) → player` | Returns the framework player object. |
| `GetJob` | `(source) → job` | Returns the player's current job table. |
| `IsOnDuty` | `(source) → boolean` | Checks if the player is on duty. |
| `GetCitizenId` | `(source) → string` | Returns the player's unique identifier. |
| `ReportError` | `(code, message)` | Logs a formatted error to console and optionally to Discord. |
| `SendDebug` | `(category, title, message, debugType, fields?)` | Sends a debug log to console and Discord webhook (only when `Config.Debug = true`). |

---

### EditTable.Functions.ErrorCodes (Server)

| Code | Constant | Meaning |
| :--- | :--- | :--- |
| `#ERR-00001` | `DATABASE_ERROR` | SQL query failed. |
| `#ERR-00002` | `PLAYER_NOT_FOUND` | Player object could not be resolved. |
| `#ERR-00003` | `INSUFFICIENT_FUNDS` | Not enough money for the operation. |
| `#ERR-00004` | `INVALID_JOB` | Job name doesn't match any configured job. |
| `#ERR-00005` | `ITEM_NOT_FOUND` | Inventory item not found. |
| `#ERR-00006` | `PERMISSION_DENIED` | Grade/rank insufficient for the action. |
| `#ERR-99999` | `UNKNOWN_ERROR` | Fallback for uncategorized errors. |

---

## 🚔 Server — LSPD Module (`editable/server/lspd.lua`)

All police-specific logic: player actions, database queries, and generator functions.

---

### EditTable.LSPD.Functions

#### `GenerateSSN()`

Generates a unique Social Security Number for citizens (6-digit numeric by default).

```lua
EditTable.LSPD.Functions.GenerateSSN = function()
    return tostring(math.random(100000, 999999))
end
```

#### `GenerateVIN()`

Generates a unique Vehicle Identification Number (12-character alphanumeric).

```lua
EditTable.LSPD.Functions.GenerateVIN = function()
    local charset = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    local str = ""
    for i = 1, 12 do
        local rand = math.random(#charset)
        str = str .. string.sub(charset, rand, rand)
    end
    return str
end
```

#### `GenerateWeaponSerial()`

Generates a weapon serial number using `Config.WeaponSerialization` settings.

```lua
EditTable.LSPD.Functions.GenerateWeaponSerial = function()
    local config = Config.WeaponSerialization
    if not config or not config.Enabled then return "UNKNOWN-" .. os.time() end
    -- Builds: "{Prefix}-{Format}" e.g. "SN-A1B2-C3D4"
end
```

#### `HandleSentence()`

Called when a full sentence (fine + jail) is applied. Handles both online and offline players.

```lua
EditTable.LSPD.Functions.HandleSentence = function(source, citizenId, fine, jailTime, reason, authorName)
    -- 1. Deduct fine from bank (online or offline DB update)
    -- 2. Add fine to police society account
    -- 3. Send to jail (online) or write to jail_sentences (offline)
    return true
end

-- Backward-compatible alias
function LSPD_HandleSentence(...)
    return EditTable.LSPD.Functions.HandleSentence(...)
end
```

---

### EditTable.LSPD.Events

| Function | Parameters | Description |
| :--- | :--- | :--- |
| `JailPlayer` | `(source, targetSource, citizenId, time)` | Sends a player to jail. Handles online and offline scenarios. |
| `FinePlayer` | `(source, targetSource, citizenId, amount, reason)` | Deducts a fine from the player's bank and deposits into the police society. |
| `GrantLicense` | `(source, targetSource, citizenId, type)` | Grants a license via MDT DB and framework metadata. |
| `RevokeLicense` | `(source, targetSource, citizenId, type)` | Removes a license from MDT DB and framework metadata. |
| `TransferVehicle` | `(source, plate, targetCitizenId)` | Transfers vehicle ownership in the database. |

<details>
<summary><b>📋 JailPlayer — Full Signature</b></summary>

```lua
EditTable.LSPD.Events.JailPlayer = function(source, targetSource, citizenId, time)
    if targetSource and targetSource > 0 then
        -- Online: uses Bridge.JailPlayer (auto-detects xr-jail, qb-prison, esx_jail)
        exports['xr-jail']:JailPlayer(targetSource, time)
        Bridge.Notify(targetSource, string.format(Locales.lspd.sent_to_jail, time), 'error')
    else
        -- Offline: insert into jail_sentences table
        MySQL.insert(
            'INSERT INTO jail_sentences (citizenid, time_remaining, in_jail) VALUES (?, ?, ?) ON DUPLICATE KEY UPDATE time_remaining = ?, in_jail = ?',
            { citizenId, time, 1, time, 1 })
    end
end
```

</details>

<details>
<summary><b>📋 FinePlayer — Full Signature</b></summary>

```lua
EditTable.LSPD.Events.FinePlayer = function(source, targetSource, citizenId, amount, reason)
    if targetSource and targetSource > 0 then
        if Bridge.RemoveMoney(targetSource, 'bank', amount) then
            Bridge.AddMoney(EditTable.LSPD.JobName, 'bank', amount)
            Bridge.Notify(targetSource, string.format(Locales.lspd.received_fine, amount, reason), 'error')
            return true
        else
            Bridge.Notify(source, Locales.system.no_money, 'error')
            return false
        end
    else
        -- Offline: creates a phone invoice
        MySQL.insert('INSERT INTO phone_invoices (citizenid, amount, society, sender, title) VALUES (?, ?, ?, ?, ?)',
            { citizenId, amount, EditTable.LSPD.JobName, 'Police Department', 'Fine: ' .. reason })
        return true
    end
end
```

</details>

---

### EditTable.LSPD.Queries

| Function | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `SearchCitizen` | `(query)` | `table[]` | Searches players by name or citizen ID. Returns processed list with `wanted` flag. |
| `GetCitizenDetails` | `(citizenId)` | `table\|nil` | Fetches full charinfo, job, metadata for a citizen. |
| `SearchVehicle` | `(query)` | `table[]` | Searches vehicles by plate or model name. |
| `GetVehicleDetails` | `(plate)` | `table\|nil` | Returns full vehicle record for a plate. |
| `UpdateOfficerStats` | `(citizenId, type, amount)` | `void` | Increments an officer's statistic counter via `UpdateStatistic` export. |
| `GetWarrants` | `(citizenId?)` | `table[]` | Returns active warrants (all, or filtered by citizen). Includes `firstName`/`lastName` from JOIN. |
| `IssueWarrant` | `(citizenId, reason, author)` | `number` | Inserts a new warrant into `lspd_warrants` and returns the ID. |
| `DeleteWarrant` | `(id)` | `void` | Removes a warrant by ID from `lspd_warrants`. |
| `IsCitizenWanted` | `(citizenId)` | `boolean` | Returns `true` if the citizen has any active warrants. |

> [!NOTE]
> All queries are framework-aware. The default implementation uses different SQL depending on `Config.Framework` (`'qb'`/`'qbox'` uses `players`, `'esx'` uses `users`). Override the specific query function if your schema differs.

---

## 🚑 Server — EMS Module (`editable/server/ems.lua`)

Medical-specific events and queries for the ambulance/EMS interface.

---

### EditTable.EMS.Events

| Function | Parameters | Description |
| :--- | :--- | :--- |
| `BillPatient` | `(source, targetSource, citizenId, amount, reason)` | Bills a patient. Online: deducts from bank, adds to ambulance society. Offline: creates phone invoice. |
| `RevivePlayer` | `(source, targetSource)` | Triggers a revive on the target player. |

> [!WARNING]
> `EMS_HandleTreatment` (used by the `ApplyTreatment` export) currently contains **hardcoded QB-Core calls** (`QBCore.Functions.GetPlayerByCitizenId`, `qb-banking`). If you use ESX or a different banking resource, override this function in `editable/server/ems.lua`:
>
> ```lua
> function EMS_HandleTreatment(source, citizenId, amount, treatmentName)
>     local targetPlayer = Bridge.GetPlayerByCitizenId(citizenId)
>     if targetPlayer then
>         Bridge.RemoveMoney(Bridge.GetSource(targetPlayer), 'bank', amount)
>         Bridge.Bank.AddMoney('ambulance', amount)
>     end
> end
> ```

---

### EditTable.EMS.Queries

| Function | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `SearchPatient` | `(query)` | `table[]` | Searches patients by name, SSN, or citizen ID. |
| `GetPatientDetails` | `(citizenId)` | `table\|nil` | Returns charinfo for a specific patient. |
| `UpdateOfficerStats` | `(citizenId, type, amount)` | `void` | Increments an EMS worker's statistic counter. |

---

## ⚖️ Server — DOJ Module (`editable/server/doj.lua`)

Court system logic — case management, citizen/vehicle lookups, and incident management.

---

### EditTable.DOJ.Queries

#### Case Management

| Function | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `GetPendingCasesCount` | `()` | `number` | Returns the total count of incidents in the database. |
| `GetCases` | `()` | `table[]` | Fetches all incidents with suspects, vehicles, officers, and notes. |
| `SearchCases` | `(query)` | `table[]` | Searches incidents by title or ID. |
| `CreateCase` | `(data, author)` | `number` | Creates a new incident with suspects, vehicles, and officers. Returns the ID. |
| `UpdateCaseStatus` | `(id, status)` | `boolean` | Updates the status of an incident (e.g. `'OPEN'`, `'CLOSED'`). |
| `UpdateCaseDescription` | `(id, description)` | `boolean` | Updates the description text of an incident. |
| `AddIncidentItem` | `(incidentId, type, item)` | `boolean` | Adds a suspect, vehicle, or officer to an incident. |
| `RemoveIncidentItem` | `(incidentId, type, itemId)` | `boolean` | Removes a suspect, vehicle, or officer from an incident. |
| `AddCaseNote` | `(incidentId, note)` | `boolean` | Appends a note to the incident's notes JSON. |

#### Citizen & Vehicle Lookups

| Function | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `SearchCitizen` | `(query)` | `table[]` | Enhanced citizen search with mugshot and wanted status. |
| `SearchVehicle` | `(query)` | `table[]` | Vehicle search with wanted/stolen tags from `lspd_vehicle_tags`. |
| `GetVehicleDetails` | `(plate)` | `table\|nil` | Full vehicle details including notes and image. |
| `AddVehicleNote` | `(plate, note, author)` | `boolean` | Appends a note to the vehicle's tag record. |
| `DeleteVehicleNote` | `(plate, noteId)` | `boolean` | Deletes a specific note from a vehicle. |
| `SetVehicleWanted` | `(plate, wanted)` | `boolean` | Flags or unflags a vehicle as wanted. |

#### Citizen Actions

| Function | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `AddCitizenNote` | `(citizenid, title, content, author)` | `number` | Adds a note to a citizen's record. |
| `SetCitizenWanted` | `(citizenid, reason, author)` | `number` | Issues a warrant for the citizen. |
| `AddSentence` | `(citizenid, fine, jail, offense, author)` | `number` | Records a conviction in `lspd_convictions`. |
| `GrantLicense` | `(citizenid, type, author)` | `number` | Grants a license via the DOJ path. |

#### Employee Management

| Function | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `GetEmployees` | `()` | `table[]` | Fetches all DOJ employees with duty status, callsign, avatar, and activity info. |

---

## 🏢 Server — Business Module (`editable/server/business.lua`)

Business panel logic — employee management, banking queries, and HR operations.

---

### EditTable.Business.Events

| Function | Parameters | Description |
| :--- | :--- | :--- |
| `HireEmployee` | `(source, businessName, targetSource)` | Hook called when someone is hired (stub — add your logic). |
| `FireEmployee` | `(source, businessName, citizenId)` | Hook called when someone is fired (stub — add your logic). |

---

### EditTable.Business.Queries

#### Employee Operations

| Function | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `GetEmployees` | `(jobName)` | `table[]` | Fetches all employees for a given job from the database. |
| `HireEmployee` | `(citizenId, jobName, grade)` | `boolean` | Sets a citizen's job in the database (primarily for offline hiring). |
| `FireEmployee` | `(citizenId)` | `boolean` | Sets a citizen's job to `'unemployed'`. |
| `GetPlayerJob` | `(citizenId)` | `table\|nil` | Returns the offline job data for validation. |
| `PromoteEmployee` | `(citizenId, jobName, newGrade)` | `boolean` | Updates the grade level of an employee. |
| `UpdateBadge` | `(citizenId, newBadge)` | `boolean` | Changes the callsign/badge stored in player metadata. |
| `SuspendEmployee` | `(citizenId)` | `boolean` | Toggles the `is_suspended` metadata flag. Also sets `job.onduty = false` when suspended. |
| `AddNote` | `(citizenId, newNote)` | `boolean` | Prepends a note to the employee's `business_notes` metadata. |
| `DeleteNote` | `(citizenId, noteId)` | `boolean` | Removes a note by ID from `business_notes`. |
| `GetBusinessBalance` | `(jobName)` | `number` | Gets the business bank balance via `Bridge.Bank.GetBalance()` or direct DB query (ESX). |

#### Finance & Banking

| Function | Parameters | Returns | Description |
| :--- | :--- | :--- | :--- |
| `GetRevenue` | `(jobName, days, offsetDays?)` | `number` | Total deposit sum from `bank_statements` for the period. `offsetDays` shifts the window back. |
| `GetExpenses` | `(jobName, days)` | `number` | Total withdrawal sum from `bank_statements` for the period. |
| `GetWeeklyRevenue` | `(jobName)` | `table[]` | Daily revenue breakdown for the last 7 days (for charts). Fields: `date_day`, `total`, `day_name`. |
| `GetRecentTransactions` | `(jobName, limit?)` | `table[]` | Last N transactions from `bank_statements`. Default limit: 5. |

> [!NOTE]
> Finance queries depend on the `bank_statements` table. If your banking resource uses a different table or schema, override these functions in `editable/server/business.lua`.

> [!WARNING]
> `Business_HandleBonus` (used by the `GiveBonus` export) currently contains **hardcoded QB-Core calls**. If you use ESX or a different framework, override this function:
>
> ```lua
> function Business_HandleBonus(employeeCitizenId, amount, businessName)
>     local targetPlayer = Bridge.GetPlayerByCitizenId(employeeCitizenId)
>     if targetPlayer then
>         Bridge.AddMoney(Bridge.GetSource(targetPlayer), 'bank', amount)
>         Bridge.Notify(Bridge.GetSource(targetPlayer),
>             string.format(Locales.business.bonus_received_notif, amount, businessName), 'success')
>     else
>         -- Offline: update bank column directly
>         if Config.Framework == 'qb' or Config.Framework == 'qbox' then
>             local player = MySQL.single.await('SELECT money FROM players WHERE citizenid = ?', { employeeCitizenId })
>             if player then
>                 local money = json.decode(player.money)
>                 money.bank = (money.bank or 0) + amount
>                 MySQL.update('UPDATE players SET money = ? WHERE citizenid = ?', { json.encode(money), employeeCitizenId })
>             end
>         end
>     end
> end
> ```

---

## 🌐 Client — Core Module (`editable/client/main.lua`)

The client-side bridge controls animations, UI initialization, dispatch routing, and targeting zones.

---

### EditTable.ClientEvents

| Function | Description |
| :--- | :--- |
| `OpenTablet()` | Plays the tablet pull-out animation when the MDT opens. |
| `CloseTablet()` | Stops the animation when the MDT closes. |

```lua
EditTable.ClientEvents.OpenTablet = function()
    RequestAnimDict("amb@world_human_seat_wall_tablet@female@base")
    while not HasAnimDictLoaded("amb@world_human_seat_wall_tablet@female@base") do Wait(10) end
    TaskPlayAnim(PlayerPedId(), "amb@world_human_seat_wall_tablet@female@base", "base", 8.0, -8.0, -1, 50, 0, false, false, false)
end

EditTable.ClientEvents.CloseTablet = function()
    StopAnimTask(PlayerPedId(), "amb@world_human_seat_wall_tablet@female@base", "base", 1.0)
end
```

> [!TIP]
> Want a different animation? Change the anim dict and clip name. Want no animation at all? Leave the function body empty.

---

### EditTable.Functions (Client)

| Function | Signature | Description |
| :--- | :--- | :--- |
| `GetPlayerData` | `() → table` | Returns the current player's data via the framework bridge. |
| `GetJob` | `() → table\|nil` | Returns the current player's job table. |
| `IsOnDuty` | `() → boolean` | Checks if the current player is on duty. |
| `ReportError` | `(code, message)` | Logs a client-side error with file/line info to console. |

**Client Error Codes:**

| Code | Constant | Meaning |
| :--- | :--- | :--- |
| `#ERR-10001` | `UI_ERROR` | NUI/UI rendering issue. |
| `#ERR-10002` | `CALLBACK_TIMEOUT` | Server callback exceeded 10-second timeout. |
| `#ERR-10003` | `ANIMATION_ERROR` | Animation dict failed to load. |
| `#ERR-10004` | `INVALID_PLAYER_DATA` | Player data is missing or malformed. |
| `#ERR-99999` | `UNKNOWN_ERROR` | Fallback for uncategorized errors. |

---

### Bridge Functions (Client)

| Function | Signature | Description |
| :--- | :--- | :--- |
| `Bridge.GetPlayerData` | `() → table` | Returns the current player's full data (job, metadata, items). |
| `Bridge.Notify` | `(text, type?, length?)` | Shows a notification (defaults: type=`'info'`, length=`5000`ms). |
| `Bridge.TriggerCallback` | `(name, cb, ...)` | Triggers a server callback with a 10-second timeout guard. |
| `Bridge.IsPlayerLoaded` | `() → boolean` | Returns true if the player is fully loaded (login check). |
| `Bridge.GetClosestVehicle` | `(coords) → entity` | Returns the closest vehicle entity. |
| `Bridge.GetClosestPlayer` | `() → id, distance` | Returns the closest player's ID and distance. |
| `Bridge.OpenMenu` | `(data)` | Opens a context menu (uses `ox_lib` context or `qb-menu`). |
| `Bridge.AddBoxZone` | `(data)` | Creates an interaction zone (auto-detects `ox_target`, `qbx_target`, `qb-target`). |
| `Bridge.AddGlobalPlayer` | `(options)` | Adds global player targeting options. |
| `Bridge.RemoveZone` | `(name)` | Removes a targeting zone by name. |

---

## 🚔 Client — LSPD Module (`editable/client/lspd.lua`)

Police-specific client commands and keybinds.

### Registered Commands

| Command | Keybind | Description |
| :--- | :--- | :--- |
| `/bk1` | — | Sends a Code 2 backup alert. |
| `/bk2` or `/bk` | — | Sends a Code 3 backup alert. |
| `/bk3` | — | Sends a Code 3 critical backup alert. |
| `/10-13` | — | Triggers an "Officer Down" alert to all emergency services. |
| `/mdt_dispatch` | — | Toggles the dispatch overlay panel. |
| `/panic_button` | `CTRL + L` | Sends a panic button alert (10-second cooldown). |

---

## 📚 Quick Start: Customizing EditTable

### Example 1 — Custom fine system with Discord logging

```lua
-- In editable/server/lspd.lua, replace FinePlayer:
EditTable.LSPD.Events.FinePlayer = function(source, targetSource, citizenId, amount, reason)
    local success = Bridge.RemoveMoney(targetSource, 'bank', amount)

    PerformHttpRequest('YOUR_WEBHOOK_URL', function() end, 'POST',
        json.encode({
            embeds = {{
                title       = '💰 Fine Issued',
                description = string.format('**%s** was fined $%s\nReason: %s', citizenId, amount, reason),
                color       = 16711680
            }}
        }), {['Content-Type'] = 'application/json'})

    return success
end
```

### Example 2 — Custom jail integration

```lua
-- In editable/server/lspd.lua, replace JailPlayer:
EditTable.LSPD.Events.JailPlayer = function(source, targetSource, citizenId, time)
    if targetSource and targetSource > 0 then
        exports['my-custom-jail']:SendToJail(targetSource, time)
    end
end
```

### Example 3 — Custom database driver

```lua
-- In editable/server/main.lua, replace Query:
EditTable.Events.Query = function(query, params)
    return exports['mysql-async']:mysql_fetch_all(query, params)
end
```

### Example 4 — Using the Global Logging System

```lua
exports['xr-mdt']:AddLog(
    source,
    "police",
    "Armory: Weapon Taken",
    "Took Glock 17 (Serial: #SN-A1B2-C3D4)",
    "Items"
)
```

### Example 5 — Custom tablet animation

```lua
-- In editable/client/main.lua, replace OpenTablet:
EditTable.ClientEvents.OpenTablet = function()
    -- Use a different animation
    RequestAnimDict("anim@cellphone@pre_selfie")
    while not HasAnimDictLoaded("anim@cellphone@pre_selfie") do Wait(10) end
    TaskPlayAnim(PlayerPedId(), "anim@cellphone@pre_selfie", "enter", 3.0, -3.0, -1, 49, 0, false, false, false)
end
```

### Example 6 — Custom Dispatch Handler (Forward to External CAD)

```lua
-- In editable/server/main.lua, extend SendDispatch:
EditTable.Events.SendDispatch = function(data)
    local jobs = data.jobs or { 'police' }
    local players = GetPlayers()

    for _, src in pairs(players) do
        src = tonumber(src)
        local playerJob = Bridge.GetJob(src)

        if playerJob and playerJob.onduty then
            for _, targetJob in pairs(jobs) do
                if playerJob.name == targetJob then
                    TriggerClientEvent('xr-mdt:client:newDispatch', src, data)
                    break
                end
            end
        end
    end

    -- Forward to external CAD system
    PerformHttpRequest('https://my-cad-system.com/api/dispatch', function() end, 'POST',
        json.encode(data), {['Content-Type'] = 'application/json', ['Authorization'] = 'Bearer YOUR_TOKEN'})
end
```

---

© XR-Core Systems | Professional FiveM Resources
