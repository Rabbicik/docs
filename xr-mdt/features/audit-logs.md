# 📊 Global Audit Logging System

**XR-MDT** includes a robust, centralized logging system designed to track every critical action performed within the tablet. This ensures accountability and provides a clear audit trail for server administrators.

---

## 🔍 How it Works

The logging system operates on a dual-path architecture:

1. **Database Persistence:** Every log is stored in the `mdt_logs` table for long-term historical review.
2. **Real-time Webhooks:** Logs are immediately forwarded to configured Discord channels based on the action type and faction, separated into administrative and operational webhooks.

The `AddLog` function is implemented in `server/sv_logs.lua` and is exposed as a server-side export.

---

## 🛠️ Usage for Developers

Trigger logs from any server-side script using the `AddLog` export.

### Export Signature

```lua
exports['xr-mdt']:AddLog(source, job, action_type, details, debugType)
```

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `source` | `number` | ✅ | Server ID of the player who performed the action. |
| `job` | `string` | ✅ | Faction/Job name (e.g., `'police'`, `'ems'`, `'doj'`). Used for webhook routing. |
| `action_type` | `string` | ✅ | The title/type of the action (e.g., `'Duty: Started'`, `'Case Created'`). |
| `details` | `string` | ✅ | Full descriptive details of the action. |
| `debugType` | `string?` | ❌ | Optional category for webhook routing overrides (e.g., `'Money'`, `'Items'`). |

### What Gets Logged Automatically

The `AddLog` function automatically resolves and stores:
- `citizenid` — from `Bridge.GetCitizenIdFromObject(player)`.
- `name` — `"firstname lastname"` from `Bridge.GetCharInfo(player)`.
- `rank` — player's current job grade name.

### Database Schema (`mdt_logs`)

| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | `INT` | Auto-increment primary key |
| `job` | `VARCHAR` | Faction name |
| `citizenid` | `VARCHAR` | Citizen unique identifier |
| `name` | `VARCHAR` | Player full name |
| `rank` | `VARCHAR` | Job grade name |
| `action_type` | `VARCHAR` | Action title |
| `details` | `TEXT` | Full details |
| `created_at` | `TIMESTAMP` | Timestamp |

### Examples

```lua
-- Log a fine being issued
exports['xr-mdt']:AddLog(
    source,
    "police",
    "Fine Issued",
    string.format("Officer fined %s $%s for: %s", citizenName, amount, reason),
    "Money"
)

-- Log an armory withdrawal
exports['xr-mdt']:AddLog(
    source,
    "police",
    "Armory: Weapon Taken",
    "Took Glock 17 (Serial: #SN-A1B2-C3D4)",
    "Items"
)

-- Log a report creation
exports['xr-mdt']:AddLog(
    source,
    "police",
    "Report: Created",
    "Officer created report #105 for Citizen John Doe"
)
```

---

## ⚙️ Configuration

Webhooks are configured in `configs/config.webhook.lua`.

### Webhook Routing Priority

The system routes logs based on the action type and its destination mapping:

1. **Routing Mapping:** The action type is resolved against `Config.Webhooks.Routing`. It returns:
   - `"admin"` — Administrative events (hiring, firing, code changes, deletions).
   - `"faction"` — Operational/faction events (fines, reports, armory).
   - `"both"` — Dispatched to both administrative and operational channels.
   - If not zmapowany, default is `"faction"` (defined as `["Default"] = "faction"`).
2. **Override Checks (debugType):** Checks if a specific override webhook exists under `Config.Webhooks.Types[debugType]`. If set, it will override the faction webhook.
3. **Specific Faction Webhooks:**
   - Operational logs: routes to `Config.Webhooks.Jobs[JOB_UPPERCASE].Faction`, then falls back to `Config.Webhooks.FactionMain`.
   - Administrative logs: routes to `Config.Webhooks.Jobs[JOB_UPPERCASE].Admin`, then falls back to `Config.Webhooks.AdminMain`.
4. **Main Webhook:** Final fallback to `Config.Webhooks.Main` if any of the above paths are empty or not configured.

### Example Webhook Configuration Structure

```lua
Config.Webhooks = {
    -- Primary global fallback webhook if others are not set
    Main = "https://discord.com/api/webhooks/...",

    -- Global fallback webhooks for faction logs and administrative logs
    AdminMain = "https://discord.com/api/webhooks/...",
    FactionMain = "https://discord.com/api/webhooks/...",

    -- Faction-specific webhooks.
    Jobs = {
        LSPD = {
            Faction = "https://discord.com/api/webhooks/...",
            Admin = "https://discord.com/api/webhooks/...",
        },
        EMS = {
            Faction = "",
            Admin = "",
        },
        DOJ = {
            Faction = "",
            Admin = "",
        },
        Business = {
            Faction = "",
            Admin = "",
        },
    },

    -- Webhooks for specific action types (overrides Jobs faction webhooks if set)
    Types = {
        Database = "",
        Money = "",
        Items = "",
        JobChange = "",
        Duty = "",
        Commands = "",
    },

    Routing = {
        -- Fallback routing for any log type not explicitly specified below
        ["Default"] = "faction",

        -- Administrative events (personnel, access, code changes, deletions)
        ["Hired"] = "both",
        ["Fired"] = "both",
        ["Promotion/Demotion"] = "both",
        ["Badge Changed"] = "both",
        ["Suspension"] = "both",
        ["License Issued"] = "both",
        ["Code Changed"] = "both",
        ["Report: Deleted"] = "both",
        ["Armory: Delete Kit"] = "both",
        ["Employee Note Created"] = "both",
        ["Employee Note Deleted"] = "both",

        -- Faction operational events
        ["Armory: Return"] = "faction",
        ["Armory: Collect"] = "faction",
        ["Armory: Save Kit"] = "faction",
        ["Armory: Collect Kit"] = "faction",
        ["Armory: Weapon Taken"] = "faction",
        ["Invoice Issued"] = "faction",
        ["Invoice Paid"] = "faction",
        ["Document: Created"] = "faction",
        ["Document: Updated"] = "faction",
        ["Document: Printed"] = "faction",
        ["Document: Received"] = "faction",
        ["Document: Signed"] = "faction",
        ["Document: Profile Printed"] = "faction",
        ["Duty: Started (CODE 1)"] = "faction",
        ["Duty: Break (CODE 2)"] = "faction",
        ["Duty: Ended (CODE 3)"] = "faction",
        ["Duty: Started"] = "faction",
        ["Duty: Break"] = "faction",
        ["Duty: Ended"] = "faction",
        ["Case Created"] = "faction",
        ["Medical Record Added"] = "faction",
        ["Patient Note Added"] = "faction",
        ["Citizen Lookup"] = "faction",
        ["Citizen Note Added"] = "faction",
        ["Warrant Issued"] = "faction",
        ["Jail Sentence"] = "faction",
        ["Fine Issued"] = "faction",
        ["Incident Report Created"] = "faction",
        ["Report: Created"] = "faction",
        ["Report: Updated"] = "faction",
    },

    -- Appearance & Colors for Discord Embeds
    Categories = {
        LSPD = { Color = 2123412, Title = "🚓 LSPD LOGS" },
        EMS = { Color = 15158332, Title = "🚑 EMS LOGS" },
        DOJ = { Color = 15844367, Title = "⚖️ DOJ LOGS" },
        Business = { Color = 3066993, Title = "💼 BUSINESS LOGS" },
        System = { Color = 8359053, Title = "⚙️ SYSTEM LOGS" },
    }
}
```

---

## 🖥️ NUI Log Viewer

Authorized employees can view logs directly within the MDT:

* Navigate to the **Logs** tab in any tablet.
* Logs are fetched via a server callback returning the last 200 entries for the job.
* Search by name, action type, or details.
* Filter results in real-time.

> [!NOTE]
> By default, the Logs tab is visible to higher ranks. Permissions are controlled via `logs_view = true` in the respective Grades tables.

---

© XR-Core Systems | Professional FiveM Resources
