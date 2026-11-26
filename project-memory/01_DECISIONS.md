# 01 — Vareindeks v4 accepted design defaults (provisional until implemented)

| ADR key | Decision | Rationale / change policy |
| --- | --- | --- |
| D-001 | Separate Django/DRF/PostgreSQL control plane | Single deployable, light operations, Atta independent |
| D-002 | Agent package uses outbound HTTPS; independent push ingestion | Runs from low-powered Linux, WSL/VPS, no inbound ports |
| D-003 | Searchable ProductIdentity created for **every** valid StoreProduct | Product coverage beats precision; no orphan/pseudo listing results |
| D-004 | ProductIdentity UUID is stable; canonical valid GTIN nullable UNIQUE | Allows no-GTIN products and cross-store grouping |
| D-005 | StoreProduct UUID, source external key and mandatory ProductIdentity FK | Reimport idempotency and history survive corrected GTIN |
| D-006 | Current price on StoreProduct; append-only PriceHistory FK to StoreProduct | No redundant Offer model, complete price timeline |
| D-007 | Source fields including reported_gtin never overwritten | Source corrections are auditable and reversible |
| D-008 | BlockedStoreGTIN `(store_id,gtin)` prevents automatic join | Wrong GTIN stays visible but cannot contaminate other Stores |
| D-009 | Manual StoreProductIdentityOverride wins over block/automatic GTIN | Narrow and persistent correction, no global GTIN equivalence |
| D-010 | Auto-import all valid rows, anomalies suppress destructive retirement only | No routine human approval queue |
| D-011 | One Store is independently priced seller, no physical inventory sites | More sellers are represented by more Stores |
| D-012 | PostgreSQL search + trigram first | No mandatory Redis/Celery/OpenSearch/S3 |
| D-013 | Exact GTIN and facets outrank text popularity; ranking cannot change identity | Useful fuzzy search without identity corruption |
| D-014 | Search telemetry aggregated and replay-bounded; licensed external priors | Privacy, gaming resistance and cold-start priors |
| D-015 | Installed allowlisted connectors; server may not distribute executable code | Agent security boundary |
| D-016 | Atta maps Store IDs itself and owns consumer identity/bonus functionality | Reusable REST service, no cross-repo coupling |
| D-017 | Product historical minimum is computed for **currently linked StoreProducts**, with coverage gaps | Simple transparent v1 query semantics; raw history never rewritten |
| D-018 | Documentation, ADR, PR evidence and project memory are versioned in repo | No invisible chat memory, tasks done only after merge |

**Accepted after v4 (24 November 2026)**

- Working name **Vareindeks**, standalone repo; no shared Atta database or user system.
- Derived `CurrentLowestPrice` and **event-based** `LowestPriceHistory` with source Store ID, StoreProduct ID and copied historical Store name.
- 24-month price histories with required boundary checkpoint; raw feeds 14 days, verbose logs 90 days, raw ranking 7 days, aggregate ranking 90 days default. Safe StoreProduct retirement after >=180 days and no dependent retained records, not because of partial feeds.
- Source of truth remains StoreProduct plus per-store PriceHistory; derived caches are repairable.

**Do not silently revert v4 to v3.1.** Breaking change to identity model/API requires a separate reviewed ADR and updated tests, backlog and contracts. Open empirical choices: source licenses, exact transaction size, perf budget, index plan, ranking coefficients, matching without GTIN beyond safe rules.