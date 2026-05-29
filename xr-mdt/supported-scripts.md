# Supported Scripts

**XR-MDT** dynamically embraces a highly versatile bridge architecture in `editable/` to conform natively with your stack.

---

## 🛠️ Framework Support

| Framework | Resource | Status |
| :--- | :--- | :--- |
| **Qbox** | `qbx_core` | ✅ Full — Uses `exports.qbx_core:GetPlayer()`, `CreateUseableItem()` |
| **QB-Core** | `qb-core` | ✅ Full — Uses `QBCore.Functions.*`, `QBCore.Commands.Add()` |
| **ESX Legacy** | `es_extended` | ✅ Full — `ESX.GetPlayerFromId()`, `RegisterUsableItem()`, server callbacks |

> [!NOTE]
> Framework is detected automatically at startup. Priority: `qbx_core` → `qb-core` → `es_extended`. You can override this with `Config.Framework = 'qb'` in `config.main.lua`.

---

## 💼 Inventory Support

| Inventory | Resource | Integration Level |
| :--- | :--- | :--- |
| **Ox Inventory** | `ox_inventory` | ⭐ Best — Uses `registerHook('useItem')` to prevent item consumption; supports item metadata for documents and bodycam states |
| **QB Inventory** | `qb-inventory` | ✅ Full — Via `QBCore.Functions.CreateUseableItem()` |
| **Quasar Inventory** | `qs-inventory` | ✅ Full — Overlapped against QB inventory structure |
| **Other** | — | Via manual item registration in `sv_items.lua` |

> [!TIP]
> When `ox_inventory` is detected, the Bridge automatically uses `OxInventory:GetItemCount()`, `AddItem()`, and `RemoveItem()` instead of framework-specific methods. This overrides all other inventory methods.

---

## 💵 Finance / Banking Integrations

The `Bridge.Bank` in `editable/server/main.lua` auto-detects your banking resource:

| Resource | Framework | Detection Method |
| :--- | :--- | :--- |
| `Renewed-Banking` | All | `GetResourceState('Renewed-Banking')` |
| `xr-bank` | All | `GetResourceState('xr-bank')` |
| `qbx_management` | Qbox | `GetResourceState('qbx_management')` |
| `qb-banking` | QB | `GetResourceState('qb-banking')` |
| `qb-management` | QB | `GetResourceState('qb-management')` |
| `esx_addonaccount` | ESX | Society accounts via `GetSharedAccount()` |

> [!IMPORTANT]
> All banking transactions (fines paid via LSPD, medical bills via EMS, bonuses via Business) are automatically recorded in the `bank_statements` table. This powers the BizPad financial dashboard. Make sure this table exists in your database.

---

## 🚔 Jail Integrations

The `Bridge.JailPlayer()` function in `editable/server/main.lua` auto-detects your jail resource:

| Resource | Framework | Method |
| :--- | :--- | :--- |
| `xr-jail` | QB / Qbox | `exports['xr-jail']:JailPlayer(source, time)` |
| `qb-prison` | QB | `TriggerEvent('qb-prison:server:SetJailStatus', source, time)` |
| `esx_jail` | ESX | `TriggerEvent('esx_jail:sendToJail', source, time * 60)` |

For a custom jail resource, override `EditTable.LSPD.Events.JailPlayer` in `editable/server/lspd.lua`:

```lua
EditTable.LSPD.Events.JailPlayer = function(source, targetSource, citizenId, time)
    if targetSource and targetSource > 0 then
        exports['my-custom-jail']:SendToJail(targetSource, time)
    end
end
```

---

## 🎯 Targeting System (ox_target / qb-target)

Used for physical interaction zones (printer zones, armory zones etc.):

| Resource | Priority |
| :--- | :--- |
| `ox_target` | 1st choice |
| `qbx_target` | 2nd choice |
| `qb-target` | 3rd choice |

The `Bridge.AddBoxZone()` function handles all three automatically. No configuration needed.

---

## 🛰️ Optional Synergies

| Resource | Integration |
| :--- | :--- |
| **XR-HUD** | HUD scaling adapts around MDT; dispatch syncs to HUD status. |
| **XR-Jail** | Native jail integration via export `JailPlayer`. |
| **FiveManage** | Mugshot uploads via `Config.FivemanageToken`. |

---

> [!NOTE]
> Have a bespoke architecture you need integrated?
> Safely configure all events and integrations via the `editable/` folder without breaking the escrow. Navigate to the [EditTable Reference](api/editable.md) for a complete API breakdown.
