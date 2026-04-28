# TMCE – Tu Módulo de Comercio Exterior

Pre-flight planning repo for a web application that helps Argentine importers manage the full import cycle: from purchase orders and supplier invoices through to nationalized unit cost calculation, stock management, and sales.


## Core idea

The hardest and most valuable part of the system is M5 (Import Cost Engine): given a set of purchase orders and invoices, it computes the nationalized unit cost per product using Argentine customs formulas (CIF, duties, TE, antidumping, IVA, gastos locales proration). Everything else is workflow around that calculation.

Architecture will follow Ports & Adapters in Python, with the domain layer (entities + cost engine) built and tested independently of any UI or delivery mechanism.
