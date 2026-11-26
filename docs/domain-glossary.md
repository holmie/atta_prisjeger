# Domain Glossary

This glossary summarizes the accepted Vareindeks domain model. The authoritative requirements are in `project-memory/00_CONTEXT.md`, `project-memory/01_DECISIONS.md`, `project-memory/02_INVARIANTS.md`, and `contracts/lowest-price.md`. This document does not introduce new models or override those contracts.

## Core models

### Store

One independently priced seller, not a physical branch or inventory location. A Store may have multiple DataSources and StoreProducts.

### DataSource

A feed or connector belonging to exactly one Store. Its source identifier and an external product key identify repeat observations from that source.

### ProductIdentity

The canonical, searchable product identity shared by StoreProducts that are legitimately matched.

- Stable UUID primary key; canonical **valid GTIN is nullable and unique**, never the primary key.
- Variant metadata helps distinguish products.
- Every structurally valid imported StoreProduct has a ProductIdentity, including those with missing or invalid GTIN. A provisional identity is searchable.
- Audited overrides, merges, splits and relinks may change canonical identity data and StoreProduct membership. They do not rewrite source claims or source price history.
- A blocked `(store_id, reported_gtin)` prevents **automatic** matching for that Store only. A persistent audited manual StoreProduct-to-ProductIdentity override takes precedence.

ProductIdentity represents **canonical identity state**, not authoritative per-seller price observations.

### StoreProduct

One source listing/offer persisted as a StoreProduct; no additional Offer or Listing model is introduced.

- Stable UUID primary key and unique `(source_id, external_key)` pair, reused on reimport.
- Mandatory `product_identity_id` foreign key (never NULL).
- Explicit `store_id` foreign key and `source_id` foreign key; `StoreProduct.store_id` must identify the same Store as its DataSource.
- Original reported GTIN/SKU, title, URL and other source evidence retained without replacing them with canonical inferences.
- Current **nullable Decimal price** and **nullable text currency**; missing/invalid price is not zero.
- Source status, availability and freshness information as supported by the feed.

StoreProduct is the source of the **current reported price for that seller**, even when its price is not the cheapest.

### PriceHistory

Append-only per-StoreProduct history of **initial valid prices and material price changes**, keyed to the StoreProduct UUID.

- Ingestion writes price events atomically and idempotently, including changes at sellers that are not cheapest.
- Unchanged price polls refresh freshness but do **not** duplicate price events.
- Availability-only transitions are **not** material price-change events in PriceHistory (I-010). Availability can independently affect the derived minimum and its coverage-gap timeline (I-021).
- Merge, split and relink never delete or rewrite the underlying StoreProduct price events.

StoreProduct and PriceHistory are the authoritative **per-seller price data**. Lowest-price projections cannot replace them.

## Derived lowest-price projections

Both projections are per **ProductIdentity**, calculated from eligible StoreProducts currently linked to that identity. They are repairable derivatives, not primary price records.

### CurrentLowestPrice

A zero-or-one-per-ProductIdentity cache of the current eligible lowest price across linked sellers.

- Stores the **derived price amount and currency**, winning `store_id`, winning `store_product_id`, and `store_name_snapshot`.
- Refreshes when an observed price change, availability/eligibility change, or identity merge/split/relink can change the current result. Recompute affected identities even where a previous cached winner may no longer be linked.
- If no defensible eligible result exists, do not fabricate a zero price or guaranteed cheapest offer.
- Stale/unknown availability, unknown shipping and mixed currencies cannot be advertised as a guaranteed cheapest total (I-020).

### LowestPriceHistory

An event-based, derived timeline of **material changes to the historical minimum**, including changes to price amount, winning seller, and availability/coverage gaps.

- Each applicable lowest-price event records its **derived amount and currency** (or a gap/unknown state), winning `store_id`, `store_product_id`, `store_name_snapshot`, and effective time.
- Known prices may carry forward only within supported observation/availability coverage. Do not infer prices before first observation or across known unavailable/unknown gaps.
- There is **no daily insert** solely because another day passed with an unchanged minimum.
- Historical results are recomputed from **currently linked StoreProducts** and their preserved source observations (D-017), within the retained horizon. A merge, split or relink can **insert, replace or remove earlier derived events even if today's winner and amount do not change**.
- Availability-only changes may switch the winner or create/close a coverage gap **without inserting a PriceHistory price event**.
- Retain 24 months of material events with the required pre-cutoff checkpoint. Preserve historical seller IDs, StoreProduct IDs and name snapshots without relying on today's Store name.

Rebuilding either derived projection never modifies raw per-seller prices or PriceHistory.

## Relationships

```text
Store (1) ----------> (N) DataSource
Store (1) ----------> (N) StoreProduct    [same Store as its DataSource]
DataSource (1) -----> (N) StoreProduct
ProductIdentity (1) -> (N) StoreProduct
StoreProduct (1) ---> (N) PriceHistory
ProductIdentity (1) -> (0..1) CurrentLowestPrice
ProductIdentity (1) -> (N) LowestPriceHistory
```

The derived projections also retain references to the **winning** Store and StoreProduct and a seller-name snapshot; they are not children of an individual PriceHistory event.

## Changes and retention

| Input/change | Source price history | Derived projections |
| --- | --- | --- |
| Initial valid price or material price change | Append an idempotent PriceHistory event | Re-evaluate affected current/historical minima |
| Unchanged price poll | No new price event; refresh freshness | No new minimum event merely for the poll |
| Availability/eligibility transition without price change | No PriceHistory price event | Refresh current result; update winner/gap events when the historical result changes |
| Merge, split or relink | Preserve StoreProduct UUIDs and raw history | Recompute affected identities' current cache **and retained historical timelines**, regardless of whether the current winner changes |

Raw feeds default to 14 days; verbose logs 90 days; raw ranking events 7 days and aggregates 90 days. Incomplete feeds never trigger destructive bulk retirement. Safe StoreProduct retirement requires at least 180 days and no dependent retained history (I-028/I-029).

## Planned regression coverage

See [negative-test.md](negative-test.md) for **unexecuted** scenarios demonstrating preservation of a losing seller's price history, historical recomputation after relink, and availability-only transitions. They are specifications, **not evidence of tests having passed**.
