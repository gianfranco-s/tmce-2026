# TMCE – MVP Plan

## Goal
Web app for Argentine importers: enter purchase orders + invoices → compute nationalized unit cost → manage stock and sales.

## Modules

### PoC — Import side (M1–M7)

| # | Name | Description |
|---|------|-------------|
| M1 | Products | CRUD with customs tax rates (duties, TE, IVA, antidumping, impuestos internos). Rates default from category, overridable per product. List shows pending POs and current stock. |
| M2 | Suppliers | Razón social, CUIT, IVA condition, currency, payment terms, rubro. Rubro drives whether the supplier gets POs (product) or direct invoices (services). Current account per supplier (invoices + payments + running balance). |
| M3 | Purchase Orders | Orders to foreign suppliers: auto-number, 5 logistics dates, multi-product lines, PDF export in English. Linkable to supplier's foreign invoice number once received. |
| M4 | Supplier Invoices | Full Argentine tax aperture (IVA 10.5/21/27, IVA_adic, Ganancias, IIBB, Derechos, TE, etc.). Each invoice stores its own exchange rate. Auto due-date from supplier payment terms. NC/ND supported. |
| M5 | Import Cost Engine | Import Dossier groups 1+ POs + selected invoices → cost calculation → confirmation produces an immutable unit-cost snapshot per product. See [COST-ENGINE.md](COST-ENGINE.md). |
| M6 | Stock | Stock entry from a confirmed dossier. Units stored as lots linked to dispatch number — required for sales invoice traceability. |
| M7 | Import Budget | Pre-decision tool: select products, enter estimated qty/price/freight/local costs → runs cost engine with estimates → shows projected unit cost and total outlay before committing to a PO. |

### MVP — Full product (M1–M10)

| # | Name | Description |
|---|------|-------------|
| M8 | Sales | Customer CRUD (razón social, CUIT, IVA condition, comercial). Sales orders with backorder detection. Customer invoices including dispatch number per line (ARCA requirement), NC/ND, customer current account. |
| M9 | Reports | Import cost by product/date, pending and completed POs, spend by rubro/period, backorder drill-down, billing, supplier and customer balances. |
| M10 | ARCA Billing | Takes confirmed sales invoices from M8 and formats them for electronic submission to ARCA. MVP: manual generation flow. Full automated integration (requires AFIP certificate) is post-MVP. |

## Key Risks
- Each invoice line needs its own stored exchange rate
- Antidumping has two modes (fixed × qty vs pct × CIF) — flagged per product
- Costo vs Desembolso split is the core accounting invariant — see COST-ENGINE.md
- Sales invoices must pre-format for ARCA even before integration is built
- M9 is not built in the PoC but the data model must be designed with reports in mind from day one — missing fields or lost intermediate values cannot be retrofitted without breaking historical accuracy
