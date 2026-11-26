# 00 — Project context

## Product

Vareindeks is a NEW standalone, lightweight product/price API and jobs control plane (Django/DRF/PostgreSQL 16), with outbound-only Python agents and source-scoped push ingest. Atta is an independent `/v1` consumer; there is NO Atta code/shared database, consumer login, personal price alert or bonus engine.

## Non-negotiable model

- `Store` (one independently priced seller, no branch inventory)
- `DataSource` (feed/connector belongs to one Store)
- `ProductIdentity` (searchable UUID, nullable UNIQUE valid GTIN and variant metadata)
- `StoreProduct` (stable UUID, `(source_id, external_key)` unique, Store FK, **ProductIdentity FK NOT NULL**, reported GTIN/SKU retained, current Decimal price nullable, URL/status)
- `PriceHistory` (append-only material changes referencing StoreProduct UUID, initial price event, no duplicate unchanged polls)

There is NO `Listing` model, `Offer` model, link join model or standalone unmatched search result. Missing/bad GTIN creates a provisional ProductIdentity; two stores with same accepted GTIN share one canonical ProductIdentity, and source keys preserve identity over time.

## Corrections

- `(store_id, reported_gtin)` block prevents **automatic** GTIN joining only for that store; retain source data, keep StoreProduct searchable via a provisional ProductIdentity
- Explicit StoreProduct-to-ProductIdentity manual override wins, persists across reimport, is auditable
- Merge/split/relink preserve StoreProduct UUIDs and raw price history. No fuzzy auto-merge required for import.

## Operations

- Stream and automatically publish every structurally valid input row without routine human approval; isolate row errors, avoid destructive retirement on incomplete/anomalous feeds.
- Job queue, scheduling, logs, leases and agent/push ingestion are observable through admin.
- Permissions and source rights matter; never attempt to evade 403/429 by IP rotation.
- Search returns ONLY ProductIdentity objects, including those with NULL GTIN; price comparison derives from linked StoreProducts; popularity boosts only relevant matches.
- No mandatory Redis/Celery/OpenSearch/S3.

## Current/historical cheapest

- derived `CurrentLowestPrice` (one-to-one cache) and `LowestPriceHistory` (event-based winner changes) retain Store UUID, StoreProduct UUID and store_name_snapshot
- No daily insert if winner/amount unchanged
- Retain 24 months of material price events plus required pre-cutoff checkpoints
- Logs/raw/ranking expire, incomplete feeds never bulk delete
- Read contracts/lowest-price.md when relevant