# 📄 Professional A4 Document Engine

**XR-MDT** features a state-of-the-art document editor designed to replicate real-world legal and administrative workflows. From police reports to business contracts, the A4 engine provides a premium editing experience.

---

## 🎨 Core Features

* **Fixed A4 Layout:** Documents are rendered in a standard A4 ratio, ensuring they look professional when viewed or printed.
* **Automatic Pagination:** Content fluidly flows across pages. When a page is full, a new one is automatically generated.
* **Dynamic Media:** Seamlessly insert images (`.png`, `.jpg`, `.webp`) directly into the document via URL.
* **Magnetic Snapping:** Images and signature slots snap to the center and margins for perfect symmetry.
* **Watermarking:** Add per-faction watermarks that sit behind the content for official branding.
* **Handwritten Signatures:** Interactive slots that players can sign, triggering a handwriting animation and embedding their name, citizenid, and timestamp permanently.
* **Real-time Sync:** Signatures are synchronized in real-time — if one person signs, everyone currently viewing the document sees it update immediately.

---

## 🛠️ How to Use the Editor

### 1. Creating a Document

1. Navigate to the **Documents** tab in your tablet.
2. Click **"New Document"** or select a **Template**.
3. Use the toolbar to format text, add headers, or insert images.

### 2. Inserting Images

1. Click the **Image Icon** in the toolbar.
2. Provide a direct URL to the image (e.g., from Discord CDN or Imgur).
3. Drag the image to position it. Use the snapping guides for alignment.

### 3. Adding Signature Slots

1. Click the **Signature Icon**.
2. A placeholder slot will appear in the document.
3. Position it where you want the player to sign.
4. When a player clicks the slot, they sign it — the slot is then locked with their name, citizenid, and the current date/time.

---

## 💾 Inventory & Persistence

### Claiming a Document

When you click **"Save & Print"**, you select a physical printer location from `Config.Printers`. After a short delay, the document becomes "claimable" at that location.

* Claiming the document gives the player a **`document`** item.
* This item contains all metadata embedded in the item's metadata field:
  - `doc_id` — Database ID in `mdt_documents`.
  - `title` — Document title.
  - `html_content` — Full rendered HTML.

### Using / Viewing

* Any player with the `document` item in their inventory can **"Use"** it.
* This opens the **Document Viewer** — a standalone, immersive NUI layer.
* The viewer is read-only except for unsigned signature slots.
* Signatures propagate to all current viewers instantly via the server.

### Database Storage

| Table | Purpose |
| :--- | :--- |
| `mdt_documents` | Stores all created documents and their current HTML state |
| `mdt_templates` | Stores reusable document templates |

---

## ⚙️ Configuration

### Printer Setup

Configure physical printer locations in `configs/config.main.lua`:

```lua
Config.Printers = {
    {
        Coords   = vector3(447.88, -973.86, 30.68),
        Label    = 'LSPD Network Printer',
        Distance = 5.0
    },
    {
        Coords   = vector3(296.31, -592.43, 43.29),
        Label    = 'Sheriff Office Printer',
        Distance = 5.0
    }
}
```

### Item Registration

The `document` item must be registered correctly in your inventory:

**Ox Inventory (recommended):**
```lua
['document'] = {
    label = 'Printed Document',
    weight = 10,
    stack = false,
    close = true,
    client = {
        export = 'xr-mdt.document'  -- Opens via client-side export
    }
}
```

**QB / ESX (via InventoryAutoRegister):**
The item is auto-registered when `Config.InventoryAutoRegister = true` and `Config.Items.document = 'document'` is set.

### Signature Style

The visual style of signed slots can be customized by modifying the signature finalization handler in the core (escrowed). The metadata fields available are: `signed_by` (name), `citizenid`, `signed_at` (timestamp).

---

## 🔧 Technical Notes

> [!WARNING]
> The document system requires `mdt_documents` and `mdt_templates` tables to exist. These are created by importing the SQL from the `INSTALL/` folder.

> [!TIP]
> For the best document item experience with metadata support, use **ox_inventory** with the `client = { export = 'xr-mdt.document' }` syntax. This ensures the item opens via the client export rather than a server-side usable item hook, which is required to pass item metadata to the viewer.

> [!NOTE]
> Do NOT use `backdrop-filter` CSS in any NUI modifications. FiveM's Chromium engine renders it as a black box. Use standard `rgba()` colors instead.

---

© XR-Core Systems | Professional FiveM Resources
