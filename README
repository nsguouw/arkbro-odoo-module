# Arkbro ERP Module — Odoo 19 Steel Fabrication Customization

A custom Odoo 19 Enterprise module built during a co-op at Arkbro Industries to solve a material traceability problem in a steel fabrication workflow. The module links supplier material certifications (Mill Test Reports) to individual inventory lots throughout purchasing, receiving, and manufacturing — ensuring every piece of steel can be traced back to its quality documentation.

## Problem

Steel fabrication requires certifications (Mill Test Reports) that prove material grade and composition. Out of the box, Odoo had no way to:
- Associate MTRs with specific inventory lots
- Track certifications through the receiving workflow
- Prevent mismatched certifications from being assigned to the wrong material
- Reuse historical purchase order lines without manual rebuilding

## What was built

### Material Test Report (MTR) System
- Custom `x_mtr` data model storing material certifications with unique auto-sequenced IDs
- Many-to-many relationships linking MTR records to purchase orders, inventory receipts, and stock lots
- Domain filtering on receipt lines so warehouse staff only see valid MTRs matching the product and receipt in context
- Boolean computed flag (`x_studio_for_receipt`) validating MTR-product-receipt matching before assignment

### Lot-as-Piece Inventory Architecture
- Each inventory lot represents a unique physical piece of steel, tracked by length rather than unit count
- `Cuttable` product category with computed cut-length field to assist manufacturing order component selection
- Custom UoM definitions (ft, 10ft, 20ft, 40ft) for bulk purchasing with automatic inventory conversion
- Length-based stock filter for inventory reporting

### Purchase Order Import Lines
- Custom "Import Lines" tab on purchase orders allowing buyers to clone historical PO lines instead of rebuilding recurring orders from scratch
- Python `.copy()` logic to duplicate records cleanly without cross-document data contamination
- XML/XPath view overrides to surface hidden vendor and arrival date fields in the selection UI

### Automation & Workflow
- Server-side automation rules for MTR naming sequences, projected arrival date sync, and RFQ history logging
- Domain-filtered views across purchasing, inventory receipts, and manufacturing orders
- Automated field updates triggered on create and edit events

## Tech Stack

- **Odoo 19 Enterprise** — ERP platform
- **Odoo Studio** — rapid field and view prototyping
- **Python** — server actions, computed fields, record copy logic
- **XML/XPath** — view inheritance, UI customization, automation rules
- **PostgreSQL** — backend database

## Files

| File | Purpose |
|------|---------|
| `__manifest__.py` | Module metadata and import order |
| `ir_model.xml` | MTR data model definition |
| `ir_model_fields.xml` | Custom fields and relationships |
| `ir_ui_view.xml` | Visual layer — all view customizations |
| `ir_actions_server.xml` | Python logic for PO import, arrival time, MTR sequencing |
| `base_automation.xml` | Automation triggers |
| `ir_sequence.xml` | MTR auto-numbering (starts at 00001) |
| `product_attribute.xml` | Material standards (ASTM A572, CSA G40.21) |
| `product_category.xml` | Cuttable category definition |
| `uom_uom.xml` | Custom units of measurement |

## Installation

### Prerequisites

- Odoo 19 Enterprise
- PostgreSQL 13+
- WKHTMLTOPDF 0.12.6-1 (add to Windows PATH for report printing)
- `odoo.conf`: set `db_template = template0`

### Required Odoo Modules

Enable before installing: `web_studio`, `purchase_stock`, `mrp`

### Required Odoo Settings

**Purchase:** Purchase Agreements, Alternatives, Lock Confirmed Orders  
**Inventory:** Lots & Serial Numbers, Units of Measure, Storage Locations, Multi-Step Routes, Packages, Variants, Replenish on Order (MTO)  
**Manufacturing:** By-Products, Unlock Manufacturing Orders, Work Orders

### Installation Command

Run as Administrator in Command Prompt (adjust paths to your environment):

```
"C:\Program Files\Odoo 19.0e.20260209\python\python.exe" "C:\Program Files\Odoo 19.0e.20260209\server\odoo-bin" -c "C:\Program Files\Odoo 19.0e.20260209\server\odoo.conf" -d arkbro_test --addons-path="C:\ArkbroCustom,C:\Program Files\Odoo 19.0e.20260209\server\odoo\addons" -i studio_customization --limit-time-real=0 --stop-after-init --dev=all
```

Extract the module zip to `C:\ArkbroCustom` before running.

### Verifying Installation

Check the log file for errors. Two expected WARNINGs (safe to ignore):
- `'watchdog' module not installed`
- `Attachment indexation of PDF documents is unavailable`

Any ERROR should be investigated. Other WARNINGs may indicate missed setup steps.

## Key Engineering Challenges

- **MTR-lot desynchronization** — lots were not being linked to MTRs despite intended workflow; resolved by placing filtering criteria on `stock.move.line` objects rather than the master `stock.lot` table, avoiding state collisions between concurrent receipts
- **Record-copy lifecycle bug** — copying historical orders caused automation dates to reset; fixed by restructuring execution targets across Odoo's `copy()` → `create()` → automation lifecycle
- **View inheritance registry breaks** — Studio XML IDs caused "Could not find view arch definition" errors after upgrades; resolved by explicitly declaring `web_studio` as a manifest dependency and retargeting XPath to core templates
- **PostgreSQL database restoration** — resolved expiration locks and UUID conflicts in restored staging environments via CLI database overrides and Developer Mode telemetry reset