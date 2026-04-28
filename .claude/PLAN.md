# TMCE – Implementation plan

## Goal
Web app for Argentine importers: enter purchase orders + invoices → compute nationalized unit cost → manage stock and sales.

## Modules

### PoC — Import side (M1–M7)

| # | Name | Description |
|---|------|-------------|
| M1 | Products | (Paso 1) CRUD with customs tax rates (duties, TE, IVA, antidumping, impuestos internos). Rates default from category, overridable per product. List shows pending POs and current stock. |
| M2 | Suppliers | (Paso 2) Razón social, CUIT, IVA condition, currency, payment terms, rubro. Rubro drives whether the supplier gets POs (product) or direct invoices (services). Current account per supplier (invoices + payments + running balance). |
| M3 | Purchase Orders | (Paso 3) Orders to foreign suppliers: auto-number, 5 logistics dates, multi-product lines, PDF export in English. Linkable to supplier's foreign invoice number once received. |
| M4 | Supplier Invoices | (Paso 4) Full Argentine tax aperture (IVA 10.5/21/27, IVA_adic, Ganancias, IIBB, Derechos, TE, etc.). Each invoice stores its own exchange rate. Auto due-date from supplier payment terms. NC/ND supported. |
| M5 | Import Cost Engine | (Paso 5) Import Dossier groups 1+ POs + selected invoices → cost calculation → confirmation produces an immutable unit-cost snapshot per product. See [COST-ENGINE.md](COST-ENGINE.md). |
| M6 | Stock | (Paso 6) Stock entry from a confirmed dossier. Units stored as lots linked to dispatch number — required for sales invoice traceability. |
| M7 | Import Budget | (Paso 2¾) Pre-decision tool: select products, enter estimated qty/price/freight/local costs → runs cost engine with estimates → shows projected unit cost and total outlay before committing to a PO. |

### MVP — Full product (M1–M10)

| # | Name | Description |
|---|------|-------------|
| M8 | Sales | (Paso 7 — orders & invoices) Customer CRUD, sales orders with backorder detection, customer invoices with NC/ND, customer current account. Covers the commercial workflow: what was sold, to whom, and at what price. |
| M9 | Reports | (Paso 8) Import cost by product/date, pending and completed POs, spend by rubro/period, backorder drill-down, billing, supplier and customer balances. |
| M10 | ARCA Billing | (Paso 7 — facturación electrónica) Takes the invoice data produced in M8 and submits it to ARCA. Split from M8 because the commercial workflow (orders, pricing, dispatch traceability) can exist and deliver value independently of whether electronic billing is wired up. M10 requires an AFIP certificate and is the only module with an external integration dependency. |

## Effort estimates
Assumes: senior developer, AI-assisted coding, minimal-to-no UI (domain + API layer only).

| # | Name | Hours | Notes |
|---|------|-------|-------|
| — | Project scaffolding | 4–6 | Ports & Adapters structure, DB setup, test infrastructure |
| M1 | Products | 3–4 | Entities, repo, category defaulting, validation |
| M2 | Suppliers | 5–7 | Entities, repo, payment terms, current account ledger |
| M3 | Purchase Orders | 6–8 | Entities, repo, auto-number, logistics dates, PDF export |
| M4 | Supplier Invoices | 5–7 | Full tax aperture, per-invoice exchange rate, auto due-date, NC/ND |
| M5 | Import Cost Engine | 14–18 | Formula chain, proration, immutable snapshot — most complex module; must validate against Excel data |
| M6 | Stock | 3–4 | Entities, repo, lot + dispatch number linking |
| M7 | Import Budget | 3–5 | Thin layer over M5 engine with estimated inputs |
| M8 | Sales | 8–10 | Customer CRUD, orders, backorder detection, invoices, current account |
| M9 | Reports | 6–8 | Aggregation queries; low effort if data model was designed correctly |
| M10 | ARCA Billing | 10–14 | Manual flow MVP; AFIP certificate + full automation is additional |

Total PoC: **~60h** 
Total MVP: **~90h** 

M5 is the only module with high uncertainty — the formula chain is well-defined but edge cases (antidumping dual mode, impuestos internos gross-up, multi-PO dossiers) need careful testing. All other estimates are relatively firm.

## Out of scope
This plan covers the import use case only. The system should be designed with the awareness that an export workflow exists as a parallel use case — same entities (products, suppliers, invoices, cost engine) but different flow and tax logic. Export is not planned here but the architecture should not make it harder to add later.

**Rough export effort estimate:** ~50–70% of the import PoC, added on top. What is reusable: scaffolding, product and supplier entities, invoice structure, stock lots, report framework. What needs to be built new: export order flow (proforma/commercial invoice to foreign buyer), export cost engine (retenciones, reintegros — different formula chain from import), export despacho, factura E billing. The architecture investment pays off here — if Ports & Adapters is done cleanly, adding export means new domain logic and adapters without touching the import side.

## Key Risks
- Each invoice line needs its own stored exchange rate
- Antidumping has two modes (fixed × qty vs pct × CIF) — flagged per product
- Costo vs Desembolso split is the core accounting invariant — see COST-ENGINE.md
- Sales invoices must pre-format for ARCA even before integration is built
- M9 is not built in the PoC but the data model must be designed with reports in mind from day one — missing fields or lost intermediate values cannot be retrofitted without breaking historical accuracy
