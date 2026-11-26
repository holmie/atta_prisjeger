# 00 — Project Context

## Product

Vareindeks is a NEW standalone, lightweight product/price API and jobs control plane (Django/DRF/PostgreSQL 16), with outbound-only Python agents and source-scoped push ingest. Atta is an independent `/v1` consumer; there is NO Atta code/shared database, consumer login, personal price alert or bonus engine.

## Non-negotiable Model

- `Store` (one independently priced seller, no branch inventory)
- `DataSource` (feed/connector belongs to one Store)
- `ProductIdentity` (searchable UUID, nullable UNIQUE valid GTIN and variant metadata)
- `StoreProduct` (stable UUID, `(source_id, external_key)` unique, Store FK, **ProductIdentity FK NOT NULL**, reported GTIN/SKU retained, current Decimal price nullable, URL/status)
- `PriceHistory` (append-only material changes referencing StoreProduct UUID, initial price event, no duplicate unchanged polls)

## Key Properties

- Stream and automatically publish every structurally valid input row without routine human approval
- Isolate row errors, avoid destructive retirement on incomplete/anomalous feeds
- Job queue, scheduling, logs, leases and agent/push ingestion are observable through admin
- Permissions and source rights matter; never attempt to evade 403/429 by IP rotation
- Search returns ONLY ProductIdentity objects, including those with NULL GTIN
- Price comparison derives from linked StoreProducts
- Popularity boosts only relevant matches

## Operations

- No mandatory Redis/Celery/OpenSearch/S3
- No Atta coupling; no parallel Offer/Listing/search-pseudo-product model
- No mandatory manual import approval
- No local inventory-location tracking
- No unrelated refactors, arbitrary code dispatch or unapproved live scrapes