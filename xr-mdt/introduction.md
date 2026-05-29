# Introduction

Welcome to the **XR-MDT** documentation!

**XR-MDT** is a highly advanced, modular tablet system designed for modern FiveM servers. It features an unparalleled user experience, bringing Police, EMS, DOJ, and Business workflows into a single cohesive ecosystem. It is meticulously optimized, fully synchronized, and structured for maximum configuration ease.

**Current Version:** `2.0` | **Author:** `Rabbicik` | **Company:** `XR Scripts`

## ✨ Key Features

* **Multi-Job Architecture:** LSPD, EMS, DOJ, and Business interfaces integrated perfectly with faction-specific themes and color branding.
* **Multi-Faction Police Support:** Support for multiple police departments (LSPD, LSSD, BCSO etc.) within a single resource, each with independent grades and permissions.
* **WASM Bodycam System:** A performant, high-quality, client-first bodycam system without server-side constraints (no FFmpeg required).
* **Professional [A4 Document Engine](features/documents.md):** Create, share, and securely sign multi-page documents with automatic pagination, magnetic snapping, and inventory persistence.
* **Advanced GPS & Tracking:** Synchronized real-time dispatch map with seamless HUD and minimap integration. Includes tracking band support.
* **Global Audit Logging:** Every critical action is logged in the `mdt_logs` database table and forwarded to customizable Discord webhooks via the `AddLog` export.
* **Dynamic Analytics Dashboard:** Look up citizens, visualize crime metrics, and oversee operations in real time.
* **Armory System:** Full weapon armory management with kit and stock tracking (`mdt_armory_kits`, `mdt_armory_stock`).
* **Weapon Serialization:** Generate unique serial numbers and register weapon ownership in the MDT evidence database (`mdt_weapons`). See [Weapon Serialization](weapon-serialization.md).
* **Database Integrity Auditor:** On every resource start, XR-MDT automatically scans for all required SQL tables and reports missing ones in the console.
* **Fully Editable Core (EditTable Bridge):** Seamlessly modify logic via the `editable/` folder while maintaining core escrow security. See [EditTable Reference](api/editable.md).
* **Localization System (i18n):** Translate absolutely everything using intuitive `configs/locales/*.lua` files. The full locale object is sent to NUI on initialization.
* **Framework-Agnostic:** Natively supports QB-Core, Qbox (`qbx_core`), and ESX Legacy through a unified `Bridge` abstraction layer.

---

## 🧭 Navigation

Here is your guide to mastering the XR-MDT environment:

> [!TIP]
> **Getting Started**
> * [Installation Guide](installation.md)
> * [Configuration](configuration.md)
> * [How To Use](how-to-use.md)

> [!NOTE]
> **Core Features**
> * [📄 A4 Document Engine](features/documents.md)
> * [📊 Audit Logging](features/audit-logs.md)
> * [🔫 Weapon Serialization](weapon-serialization.md)

> [!NOTE]
> **Technical References**
> * [Supported Resources](supported-scripts.md)
> * [Common Issues](common-issues.md)
> * [Client API Reference](api/client.md)
> * [Server API Reference](api/server.md)
> * [EditTable Reference](api/editable.md) — Complete guide to all editable functions

> [!IMPORTANT]
> **Step-by-Step Guides**
> * [🚔 Adding a New Police Unit](guides/adding-police-unit.md)
> * [🏢 Adding a New Business to BizPad](guides/adding-business.md)
