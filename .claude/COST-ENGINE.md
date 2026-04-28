# Cost Engine – Formulas & Data Model Detail

## Key Accounting Distinction
- **COSTO TOTAL** = what goes into unit cost (FOB + duties + TE + dumping + SIM + local expenses net)
- **DESEMBOLSO TOTAL** = total cash out = Costo + recoverable taxes (IVA, IVA_Adic, Ganancias, IIBB, II, IVA on local expenses)

IVA, IVA_Adic, Ganancias, IIBB are **not costs** — they are recoverable or withholdings. Must be tracked separately.

## Formula Chain (derived from COSTOS DE IMPORTACIÓN.xlsx)

| Step | Formula |
|------|---------|
| Seguro | `(FOB_total + Flete_certificado) × 0.25%` — auto-calculated |
| CIF (Valor en Aduana) | `FOB + Flete_certificado + Seguro` |
| Derechos per product | `CIF_prod × derechos_rate` |
| TE per product | `CIF_prod × TE_rate` |
| Dumping per product | `fixed_amount × qty` OR `CIF_prod × rate` (per `antidumping_type` flag) |
| **B.I. per product** | `CIF_prod + Derechos + TE + Dumping` — base for all remaining taxes |
| IVA | `B.I. × IVA_rate` |
| IVA Adicional | `B.I. × IVA_adic_rate` (rule: 20% when IVA=21%, 10% when IVA=10.5%) |
| Ganancias | `B.I. × 0.06` |
| IIBB | `B.I. × 0.0275` |
| Base II (gross-up) | `((tasa_ef / (1 − tasa_ef)) + 1) × 1.3 × B.I.` |
| Impuestos Internos | `Base_II × tasa_nominal` |
| SIM | `total_SIM_fixed × (product_FOB / total_FOB)` — prorated by FOB share |
| Gastos locales per product | `total_gastos_locales_neto × (product_FOB / total_FOB)` — all local costs pooled, then split |
| **COSTO TOTAL per product** | `FOB + Derechos + TE + Dumping + SIM + Gastos_locales_share` |
| COSTO UNITARIO | `COSTO_TOTAL / qty` |
| DESEMBOLSO TOTAL | `COSTO_TOTAL + IVA + IVA_Adic + Ganancias + IIBB + II + IVA_on_local_expenses` |
| Honorarios Despachante | `(gastos_locales + aduana_total + FOB) × 1.1 × 0.4%`, min USD 270 |

Note: `Flete_certificado` (declared to customs for CIF) differs from the actual paid freight (a separate gastos locales invoice line).

## Data Model Detail

### `products`
- code (PK, immutable), description, currency, UoM, category_id
- derechos_rate, TE_rate, IVA_rate, IVA_adic_rate
- antidumping_type: `pct | fixed`, antidumping_value
- II_tasa_efectiva, II_tasa_nominal
- Rates default from category but are overridable per product

### `import_dossier` (Legajo)
- Groups 1+ purchase orders into one shipment
- Stores: flete_certificado, TC_despacho, dispatch_no, transport_doc, SIM_total, adjustments
- Status: open → confirmed (locked on cost confirmation)

### `invoices`
- Full tax aperture: neto, no_gravado, exento, IVA 10.5/21/27, IVA_adic 10/20, Ganancias, IIBB, Derechos, TE, II, Multa, Percepcion_IVA, Percepcion_IIBB, total
- Each invoice stores its own TC at time of entry (mandatory)
- Auto due-date calculated from supplier payment_terms + invoice date
- Linkable to dossier (optional at entry, required before cost confirmation)

### `import_cost_lines`
- Immutable snapshot created on dossier confirmation
- Stores all intermediate values per product: CIF_prod, B.I., each tax amount, gastos_locales_share, costo_total, costo_unitario, desembolso
- Never recalculated after confirmation — source of truth for unit cost and reports

### `stock_lots`
- qty, unit_cost_usd, dispatch_no (must appear on sales invoices per ARCA)
- Created from confirmed dossier; dispatch_no propagates to every sales invoice line

## Presupuesto de Importación
Reuses the same formula chain with estimated inputs instead of actuals. Build after Phase 3 cost engine is stable — it is a thin UI layer over the same calculation service.
Reference tariff sheets in the Excel (Cotizaciones de Flete, Custodia, Acarreo, Deposito Fiscal, Certificación) are the lookup tables that feed presupuesto inputs.
