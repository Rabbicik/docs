# 🏢 Adding a New Business to BizPad

This step-by-step guide walks you through adding a **new business** to the BizPad system in XR-MDT, including branding (logo, colors), permissions, and troubleshooting common problems.

---

## 📋 Prerequisites

Before you begin, make sure you have:

- ✅ A **job** defined in your framework (`qb-core/shared/jobs.lua` or ESX `jobs` table)
- ✅ A bank account for the business in your banking system (e.g. `qb-banking`, `Renewed-Banking`, `xr-bank`)
- ✅ Access to the file `configs/config.business.lua`
- ✅ (Optional) A logo image (`.png`, `.webp`, or `.jpg`)

---

## 📖 How Does BizPad Work?

BizPad is the XR-MDT module designed for **businesses and companies**. It provides:

| Feature | Description |
| :--- | :--- |
| **Dashboard** | Revenue, expenses, statistics, weekly charts |
| **Employees** | Full HR: hire, fire, promote, demote, suspend, badge assignment |
| **Invoices** | Create and send invoices to players (real-time accept/reject) |
| **Tariff** | Manage a price list / service catalog |
| **Notes** | Internal HR notes per employee |

Each business is identified by its **framework job name**. The system automatically pulls employees, grades, and data from the database.

---

## 🔧 Step 1 — Add the Job to `AllowedJobs`

Open `configs/config.business.lua` and add the job name to the `AllowedJobs` list:

```lua
Config.Business = {
    TabletCommand = 'bizpad',
    OpenKey = 'DELETE',

    AllowedJobs = {
        'mechanic',
        'cardealer',
        'realestate',
        'taxi',
        'burgershot',
        'uwucafe',
        'autoexotica',    -- ← NEW BUSINESS
    },
    -- ...
}
```

> [!IMPORTANT]
> The name **must exactly match** the job name in your framework. If `shared/jobs.lua` (QB) has `['autoexotica']`, you must write `'autoexotica'` here — not `'AutoExotica'` or `'auto_exotica'`.

---

## 🔧 Step 2 — Add Branding (Logo, Name, Colors)

Add an entry to the `JobSettings` table to give your business a custom logo, display name, and color theme:

```lua
Config.Business = {
    -- ...
    JobSettings = {
        -- ... existing entries ...

        ['autoexotica'] = {
            DisplayName = 'Auto Exotica',                    -- shown in the BizPad header
            LogoFile    = 'images/biz_autoexotica.webp',     -- path relative to web/dist/
            Colors      = {                                  -- optional accent colors
                primary     = '#8b5cf6',                     -- main accent
                primaryDark = '#6d28d9',                     -- darker variant
                bg          = 'rgba(139, 92, 246, 0.1)'     -- element backgrounds
            },
        },
    },
    -- ...
}
```

Then place your logo file at:
```
web/dist/images/biz_autoexotica.webp
```

**Supported image formats:** `.png`, `.webp`, `.jpg`

> [!TIP]
> If you don't add a `JobSettings` entry, the business will still work — it will use the framework's job label as the name and show no logo. Branding is **optional**.

---

## 🔧 Step 3 — Configure Permissions (Grades)

You have **two approaches** to configure permissions:

### Option A — Use Default Permissions

If you don't define a custom block for your business, the system automatically uses the `['default']` grades. This is the simplest option for standard businesses.

The default config provides 5 grade levels (0–4):

| Grade | Role | Key Permissions |
| :--- | :--- | :--- |
| `0` | Employee | View dashboard, create invoices, view tariff |
| `1` | Experienced | Same as grade 0 |
| `2` | Manager | + `employees_manage`, `invoice_custom`, `tariff_manage`, `admin` |
| `3` | Owner | Full access |
| `4` | Owner+ | Full access |

> [!TIP]
> If your business uses a standard 5-grade hierarchy and doesn't need special permissions — **you don't need to do anything else**. Just add it to `AllowedJobs` and optionally `JobSettings`.

---

### Option B — Define Custom Permissions Per Business

For businesses with a non-standard hierarchy, add a dedicated block in `Grades`:

```lua
Grades = {
    ['default'] = { ... },

    -- ========================================================
    -- AUTO EXOTICA — Luxury Car Dealership
    -- ========================================================
    ['autoexotica'] = {
        ['0'] = {
            label = 'Intern',
            permissions = {
                dashboard_view   = true,
                employees_view   = true,
                employees_manage = false,
                invoices_create  = false,   -- interns can't create invoices
                invoices_view    = true,
                invoice_custom   = false,
                tariff_view      = true,
                tariff_manage    = false,
                admin            = false,
            }
        },
        ['1'] = {
            label = 'Sales Associate',
            permissions = {
                dashboard_view   = true,
                employees_view   = true,
                employees_manage = false,
                invoices_create  = true,
                invoices_view    = true,
                invoice_custom   = false,
                tariff_view      = true,
                tariff_manage    = false,
                admin            = false,
            }
        },
        ['2'] = {
            label = 'Senior Sales',
            permissions = {
                dashboard_view   = true,
                employees_view   = true,
                employees_manage = false,
                invoices_create  = true,
                invoices_view    = true,
                invoice_custom   = true,    -- can set custom invoice amounts
                tariff_view      = true,
                tariff_manage    = false,
                admin            = false,
            }
        },
        ['3'] = {
            label = 'Floor Manager',
            permissions = {
                dashboard_view   = true,
                employees_view   = true,
                employees_manage = true,    -- manages employees
                invoices_create  = true,
                invoices_view    = true,
                invoice_custom   = true,
                tariff_view      = true,
                tariff_manage    = true,    -- manages tariff/price list
                admin            = false,
            }
        },
        ['4'] = {
            label = 'Owner',
            permissions = {
                dashboard_view   = true,
                employees_view   = true,
                employees_manage = true,
                invoices_create  = true,
                invoices_view    = true,
                invoice_custom   = true,
                tariff_view      = true,
                tariff_manage    = true,
                admin            = true,    -- full access
            }
        },
    },
}
```

> [!WARNING]
> **Grade keys must be strings** (`'0'`, `'1'`, `'2'`, etc.), not numbers. The system looks up grades using `tostring(job.grade.level)`. Using numeric keys will cause a permissions lookup failure and the BizPad will appear empty or broken.

---

## 🔧 Step 4 — Bank Account Setup

The BizPad dashboard displays balance, revenue, and transaction history. This data comes from your banking system. **The bank account must exist** under the job name.

### Automatically Supported Banking Systems

| Banking Resource | Supported |
| :--- | :--- |
| `qb-banking` | ✅ Auto-detected |
| `Renewed-Banking` | ✅ Auto-detected |
| `xr-bank` | ✅ Auto-detected |
| `qbx_management` | ✅ Auto-detected |
| `esx_addonaccount` | ✅ Auto-detected |

### QB-Core / QBX
The bank account is usually created automatically by `qb-management` or `qb-banking` when the job is defined in `shared/jobs.lua`. No extra steps needed.

### ESX
You **must** manually insert the society account:
```sql
INSERT INTO addon_account (name, label, shared) VALUES ('society_autoexotica', 'Auto Exotica', 1);
```

> [!CAUTION]
> If the bank account doesn't exist, the dashboard will show `$0` for balance, revenue, and expenses. This is the **#1 reason** businesses appear broken. Always verify the account exists before reporting a bug.

---

## 🔧 Step 5 — Database Tables

The BizPad relies on two database tables that are created by `xr-mdt.sql`:

| Table | Purpose |
| :--- | :--- |
| `business_invoices` | Stores all invoices (pending, paid, rejected) |
| `business_tariffs` | Stores the price list / service catalog |

> [!WARNING]
> If these tables don't exist, the invoice and tariff tabs will crash. Run the full `xr-mdt.sql` migration if you haven't already.

### Verify Tables Exist
```sql
SHOW TABLES LIKE 'business_%';
```

Expected output:
```
business_invoices
business_tariffs
```

---

## 🔧 Step 6 — Adjust Limits (optional)

At the bottom of `config.business.lua` you'll find global limits:

```lua
Config.Business = {
    -- ...
    MaxInvoicesPerDay = 50,    -- max invoices per day (per player)
    MaxBonusAmount    = 10000, -- max bonus payout amount
}
```

> [!NOTE]
> Limits are **global** — they apply equally to all businesses. Per-business limits are not currently supported.

---

## 🔧 Step 7 — Restart & Verify

```
ensure xr-mdt
```

Then log in as a player with the `autoexotica` job and type `/bizpad`.

---

## ✅ Checklist

| Step | Status |
| :--- | :--- |
| Job defined in framework (qb/esx) | ☐ |
| Added to `Config.Business.AllowedJobs` | ☐ |
| (Optional) Branding added in `JobSettings` | ☐ |
| (Optional) Logo placed in `web/dist/images/` | ☐ |
| Permissions configured (default or custom) | ☐ |
| Bank account exists in banking system | ☐ |
| (ESX) Society account inserted into `addon_account` | ☐ |
| Database tables exist (`business_invoices`, `business_tariffs`) | ☐ |
| Resource restarted | ☐ |

---

## 🔥 Troubleshooting — Common Business Issues

### BizPad opens but shows empty dashboard / $0 everywhere

**Cause:** The bank account doesn't exist or your banking resource uses a different table name.

**Fix:**
1. Verify the bank account exists in your banking system under the exact job name
2. If your banking system uses a non-standard table, override `Bridge.Bank.GetBalance` in `editable/server/main.lua`:
```lua
function Bridge.Bank.GetBalance(accountName)
    return exports['my-custom-bank']:FetchMoney(accountName)
end
```

---

### "No Access" notification when opening BizPad

**Cause:** The player's job name doesn't match any entry in `AllowedJobs`, or the permissions callback returned `nil`.

**Fix:**
1. Check the player's exact job name in F8 console or admin panel
2. Ensure it's **exactly** spelled the same in `AllowedJobs`
3. Make sure `Config.EnableBusiness = true` in `config.main.lua`

---

### Employee list is empty even though workers are assigned

**Cause:** Grade mismatch between framework and config, or the database query is filtering out employees.

**Fix:**
1. Ensure the framework job grades match the `Grades` keys in your config
2. For QB: verify that `players.job` JSON contains the correct job name
3. For ESX: verify the `users.job` column matches
4. Check F8 console for SQL errors

---

### Invoices not being created / "unknown column" error

**Cause:** The `business_invoices` table is missing or has an outdated schema.

**Fix:**
```sql
-- Run the full table creation from xr-mdt.sql, or manually:
CREATE TABLE IF NOT EXISTS `business_invoices` (
    `id` INT AUTO_INCREMENT PRIMARY KEY,
    `business_name` VARCHAR(50) NOT NULL,
    `citizenid` VARCHAR(50) NOT NULL,
    `amount` INT NOT NULL DEFAULT 0,
    `title` VARCHAR(255) DEFAULT NULL,
    `issuer` VARCHAR(100) DEFAULT NULL,
    `status` VARCHAR(20) DEFAULT 'Pending',
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS `business_tariffs` (
    `id` INT AUTO_INCREMENT PRIMARY KEY,
    `business_name` VARCHAR(50) NOT NULL,
    `label` VARCHAR(255) NOT NULL,
    `amount` INT NOT NULL DEFAULT 0
);
```

---

### Employee management actions fail silently

**Cause:** The acting player's grade is lower than or equal to the target's grade. The system prevents you from promoting/firing someone at or above your own rank.

**Fix:** This is intentional behavior. Only higher-ranked employees can manage lower-ranked ones. The owner (highest grade) can manage everyone.

---

### Finance charts show no data

**Cause:** Your banking resource doesn't write to the `bank_statements` table, which the revenue/expense queries depend on.

**Fix:** Override the finance queries in `editable/server/business.lua` to use your banking system's transaction table:

```lua
EditTable.Business.Queries.GetRevenue = function(jobName, days, offsetDays)
    local offset = offsetDays or 0
    local result = MySQL.scalar.await([[
        SELECT COALESCE(SUM(amount), 0) FROM your_custom_transactions_table
        WHERE account_name = ? AND type = 'deposit'
        AND created_at >= NOW() - INTERVAL ? DAY
        AND created_at < NOW() - INTERVAL ? DAY
    ]], { jobName, days + offset, offset })
    return tonumber(result) or 0
end
```

See the full [EditTable Business Reference](../api/editable.md#-server--business-module-editableserverbusinesslua) for all overridable queries.

---

### Logo not displaying

**Cause:** Wrong file path, wrong format, or the file is missing from `web/dist/`.

**Fix:**
1. Ensure the file is placed at `web/dist/images/biz_yourjob.webp` (or `.png`/`.jpg`)
2. Ensure the `LogoFile` path in `JobSettings` matches exactly (case-sensitive)
3. Clear your FiveM cache: `AppData/Local/FiveM/FiveM.app/data/cache`
4. Restart the resource

---

## 📝 BizPad Permissions Reference

| Permission | Description |
| :--- | :--- |
| `dashboard_view` | Access the dashboard with financials and statistics |
| `employees_view` | Browse the employee list |
| `employees_manage` | Hire, fire, promote, demote, suspend employees |
| `invoices_create` | Create invoices and send them to players |
| `invoices_view` | Browse invoice history |
| `invoice_custom` | Create invoices with a custom (freeform) amount |
| `tariff_view` | Browse the service price list |
| `tariff_manage` | Add, edit, and delete tariff entries |
| `admin` | Full administrative permissions (includes all of the above) |

---

## 🎯 Minimal Example — Adding a Business in 3 Lines

The simplest way to add a new business (no custom branding, default permissions):

```lua
-- In configs/config.business.lua:

Config.Business = {
    TabletCommand = 'bizpad',
    OpenKey = 'DELETE',

    AllowedJobs = {
        'mechanic',
        'cardealer',
        'realestate',
        'taxi',
        'burgershot',
        'uwucafe',
        'autoexotica',    -- ← ADD HERE (1 line)
    },

    -- JobSettings: not required for basic setup
    -- Grades: default permissions will apply automatically
}
```

> [!TIP]
> If your business has a standard hierarchy (0 = employee, 1 = experienced, 2 = manager, 3–4 = owner), adding it to `AllowedJobs` is all you need. Default permissions work immediately.

---

## 🎯 Full Example — Business with Custom Branding

```lua
Config.Business = {
    TabletCommand = 'bizpad',
    OpenKey = 'DELETE',

    AllowedJobs = {
        'mechanic', 'cardealer', 'realestate', 'taxi',
        'burgershot', 'uwucafe',
        'autoexotica',    -- ← 1. Add to allowed list
    },

    JobSettings = {
        -- ... existing entries ...
        ['autoexotica'] = {                              -- ← 2. Add branding
            DisplayName = 'Auto Exotica',
            LogoFile    = 'images/biz_autoexotica.png',  -- supports .png, .webp, .jpg
            Colors      = {
                primary     = '#8b5cf6',
                primaryDark = '#6d28d9',
                bg          = 'rgba(139, 92, 246, 0.1)'
            },
        },
    },

    Grades = {
        ['default'] = { ... },
        ['autoexotica'] = {                              -- ← 3. Custom permissions (optional)
            ['0'] = { label = 'Intern',        permissions = { ... } },
            ['1'] = { label = 'Sales',         permissions = { ... } },
            ['2'] = { label = 'Senior Sales',  permissions = { ... } },
            ['3'] = { label = 'Manager',       permissions = { ... } },
            ['4'] = { label = 'Owner',         permissions = { ... } },
        },
    },

    MaxInvoicesPerDay = 50,
    MaxBonusAmount = 10000,
}
```

---

© XR-Core Systems | Professional FiveM Resources
