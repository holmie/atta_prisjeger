# 03 — API Contracts

This document defines the canonical API contracts for Vareindeks.

## Core Models

### ProductIdentity
- Stable UUID
- Nullable unique canonical GTIN
- Variant metadata
- Searchable via PostgreSQL trigram index

### StoreProduct
- Stable UUID
- `(source_id, external_key)` unique constraint
- FK to ProductIdentity (NOT NULL)
- Reported GTIN/SKU retained
- Current Decimal price nullable
- URL/status information

### PriceHistory
- Append-only material changes
- References StoreProduct UUID
- Records initial valid price + material changes only
- Unchanged polls refresh freshness only

## API Endpoints

### `/v1/products`
- Returns ProductIdentity objects (including GTIN=NULL provisional objects)
- Never raw listing pseudo-items
- Searchable by GTIN, facets and text search

### `/v1/store-products`
- Returns StoreProduct objects with linked ProductIdentity
- Price comparison derived from linked StoreProducts
- Supports filtering by source_id, store_id, etc.

## Data Flow

- Source-authorized, valid records automatically publish
- Individual bad rows are isolated
- No routine import approval queue
- Job queue, scheduling, logs, leases and agent/push ingestion are observable through admin

## Security & Compliance

- All network/data work has bounded bytes, memory, time, query size and concurrency
- Explicit 403/429 respects access policy
- Agent packages use outbound HTTPS only
- Installed allowlisted connectors; server may not distribute executable code
- Remote servers never send arbitrary executable Python code