# 02 — Hard invariants (require tests and honest evidence)

| ID | Mandatory fact |
| --- | --- |
| I-001 | Every published valid StoreProduct has exactly one **non-null** ProductIdentity FK, even with missing/invalid GTIN and price. |
| I-002 | Every ProductIdentity has stable UUID; canonical GTIN is a **nullable unique string**; never the primary key. |
| I-003 | `(source_id,external_key)` uniquely and idempotently identifies one StoreProduct. `(store_id,GTIN)` is **not** unique. |
| I-004 | Two different Stores importing the same valid non-blocked GTIN resolve to one canonical ProductIdentity, with PostgreSQL race handling. |
| I-005 | StoreProduct reported GTIN, source SKU, title, URL and source evidence are not replaced by canonical normalized inference. |
| I-006 | Blocked `(store_id,raw GTIN)` prevents *automatic* linking for that Store only and never deletes its StoreProduct. |
| I-007 | An audited StoreProduct→ProductIdentity manual override persists through reimport and supersedes blocked GTIN automatic behavior. |
| I-008 | A GTIN-less StoreProduct reimport with same stable source key reuses both StoreProduct and ProductIdentity UUIDs. |
| I-009 | Price and currency are Decimal/text on StoreProduct, nullable when unknown; a missing/invalid price never becomes zero. |
| I-010 | PriceHistory references **StoreProduct** and records initial valid price + material changes only, atomically/idempotently. Unchanged polls refresh freshness only. |
| I-011 | Merge/split/relink of ProductIdentity never deletes or rewrites StoreProduct raw prices/history; dependent search/min aggregates are refreshed. |
| I-012 | Top-level `/v1/products` returns only ProductIdentity (including GTIN=NULL provisional objects), never raw listing pseudo-items. |
| I-013 | Source-authorized, valid records automatically publish; individual bad rows are isolated; no routine import approval. |
| I-014 | Partial/incomplete/anomalously shrinking feed cannot bulk deactivate absent StoreProducts; valid updates still apply. |
| I-015 | JobRun, batches, leases and price changes are crash-resilient; replay causes no duplicate state transitions. |
| I-016 | Agent/run credential, push source credential, read API key and operator sessions have noninterchangeable scopes. |
| I-017 | Remote server never sends arbitrary executable Python code; only allowlisted installed connectors, versioned configuration. |
| I-018 | No local warehouse/branch inventory and no separate Offer or Listing persistence model in v1. |
| I-019 | Exact GTIN/filters beat popularity; ranking activity and external priors do not mutate identity or raw source claims. |
| I-020 | Unknown shipping, stale/unknown availability, mixed currencies cannot masquerade as guaranteed cheapest total. |
| I-021 | Derived LowestPriceHistory records material lowest-price, winner Store ID/name and availability-gap transitions; carry forward known prices within covered intervals, never pre-observation or known unavailable gaps. |
| I-022 | Client/search logs do not require user IDs or long-lived personal event trails; bounded source caps prevent simple rank gaming. |
| I-023 | All network/data work has bounded bytes, memory, time, query size and concurrency; explicit 403/429 respects access policy. |
| I-024 | Live source rights and redistribution scope must be verified independently of syntax-valid scraper success. |
| I-025 | API `/v1` identifiers are stable, pagination/change cursors resumable, secrets not exposed in consumer responses. |
| I-026 | Task ledger DONE is human-attested actual merge SHA/PR/tests; milestone gates need independent evidence. |
| I-027 | CurrentLowestPrice cache and LowestPriceHistory are derived/repairable; historical seller store_id, store_product_id, store_name_snapshot preserved without a current Store-name JOIN. |
| I-028 | History is bounded to 24 months plus pre-cutoff checkpoint; logs/raw/rank events expire; retiring StoreProducts cannot destroy retained price history. |
| I-029 | Stable input volume plus stable activity should reach measurable bounded storage; partial feeds never trigger destructive cleanup. |
| I-030 | Core architecture is monolithic Django/DRF/PostgreSQL control plane with outbound-only agents; no mandatory broker/search cluster |

**Design-conflict policy:** halt scope change and propose ADR. Do not delete negative tests or change cards silently to make CI green.