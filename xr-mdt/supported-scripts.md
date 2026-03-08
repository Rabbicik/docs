---
title: "Supported Scripts"
description: "Frameworks, inventories, and banking systems compatible with XR-MDT."
---

**XR-MDT** is built modularly and supports many popular FiveM scripts.

## Frameworks
*   **QB-Core:** Supported with charinfo, metadata, citizenid.
*   **ESX:** Supporting legacy and newest versions, `users` and `user_licenses` tables.
*   **Qbox:** Through qbox-core bridge.

## Inventories
*   **QB-Inventory:** Standard item integration.
*   **Ox Inventory:** Supports `registerHook` and metadata (e.g., bodycam).
*   **Quasar Inventory:** Through QB-Core bridge.

## Databases
*   **OxMySQL:** Recommended SQL query engine.

## Other Dependencies
*   **Ox Lib:** For notifications, menus, and callbacks.
*   **QB-Menu:** (If using QB-Core without ox_lib).
*   **QB-Target / Ox Target:** For zone-based opening of MDT.

## Compatible Banking Systems
*   **Renewed-Banking**
*   **QB-Banking**
*   **XR-Bank**
*   **ESX Standard Society**

---

If you use custom solutions, you can easily integrate your script in the `editable/` folder.
*   [Client side bridge](api/client-side.md)
*   [Server side bridge](api/server-side.md)
