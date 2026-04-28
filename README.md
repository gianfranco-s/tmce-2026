# TMCE – Tu Módulo de Comercio Exterior

Pre-flight planning repo for a web application that helps Argentine importers manage the full import cycle: from purchase orders and supplier invoices through to nationalized unit cost calculation, stock management, and sales.


## Core idea

The hardest and most valuable part of the system is M5 (Import Cost Engine): given a set of purchase orders and invoices, it computes the nationalized unit cost per product using Argentine customs formulas (CIF, duties, TE, antidumping, IVA, gastos locales proration). Everything else is workflow around that calculation.

Architecture will follow Ports & Adapters in Python, with the domain layer (entities + cost engine) built and tested independently of any UI or delivery mechanism.

## Market research strategy (before writing code)

The goal is to validate two things: whether the pain is real, and who will pay to fix it.

**Step 1 — Find one of three profiles**
- **Importer** — feels the financial pain, has budget authority
- **Accountant (contador)** — owns the calculation today, serves multiple importers
- **Despachante** — knows the process deeply, works with many importers

Start with warm contacts. For cold outreach, LinkedIn is the channel.

**Step 2 — LinkedIn message asking for a conversation**
Keep it short. Lead with the import/comex angle — "I'm exploring a problem in the import space and would love 20 minutes of your time". The only goal of this message is to get a yes to a call.

**Step 3 — Ask one question on the call**
"How do you currently calculate the nationalized unit cost of an import?" Then stop talking. Listen for frustration, manual workarounds, and what they wish existed. Do not pitch, do not describe the product yet.

If the conversation goes well, follow up with:
- Who owns this in your org — you, your accountant, your despachante?
- What does a mistake in this calculation cost you?
- Would you pay for a tool that did this reliably? How much, per month?

**Decide after 10 conversations**
If 3 or more name a price unprompted, build the PoC. If answers are vague or the buyer is unclear, adjust scope before writing code.
