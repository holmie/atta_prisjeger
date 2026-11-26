# Product Charter: Vareindeks

## Purpose

Vareindeks is a standalone, lightweight product/price API and jobs control plane that provides price data services for retailers and consumers. It enables independent sellers to publish their pricing information through a structured API, while providing consumers with access to competitive pricing data across multiple sources.

## Actors

- **Store Owners/Sellers**: Independent retailers who provide product information via feeds or push ingestion
- **Data Consumers**: Applications that consume price data (including Atta as a consumer)
- **System Operators**: Technical personnel responsible for managing the Vareindeks system
- **Agents**: Outbound-only Python agents that fetch data from external sources

## V1 Scope

### Core Functionality
- Store product management with unique identifiers and pricing information
- Price history tracking with append-only material changes
- Product identity management with stable UUIDs and GTIN handling
- Search functionality returning only ProductIdentity objects (including provisional ones without GTIN)
- API endpoints for price data access at `/v1/products`
- Job queue, scheduling, logs, and leases observable through admin interface

### Data Model
- `Store`: Represents an independently priced seller with no branch inventory
- `DataSource`: Feed/connector belonging to one Store
- `ProductIdentity`: Searchable UUID with nullable UNIQUE valid GTIN and variant metadata
- `StoreProduct`: Stable UUID with `(source_id, external_key)` unique constraint, referencing a ProductIdentity
- `PriceHistory`: Append-only material changes referencing StoreProduct UUID

### Operations
- Automatic publishing of valid rows without routine human approval
- Error isolation for individual bad rows
- No destructive retirement on incomplete/anomalous feeds
- Crash-resilient job processing with replay safety
- Source rights and permissions enforcement
- Bounded storage and data lifecycle management (24-month price history, 14-day raw feeds, 90-day logs)

## Explicit Exclusions

### Consumer Features
- No consumer accounts or user authentication system
- No email notifications or alerting system
- No Atta reward computation or bonus engine functionality
- No personal price alerts or tracking features

### System Design
- No parallel Offer/Listing/search-pseudo-product model
- No mandatory manual import approval
- No local inventory-location tracking
- No Redis/Celery/OpenSearch/S3 dependencies
- No separate Offer or Listing persistence model in v1
- No cross-repo coupling with Atta system

### Data Handling
- No routine human approval queue for valid records
- No destructive cleanup of partial feeds
- No automatic merging/splitting of products without explicit override
- No modification of source data during import (GTIN, SKU, title, URL retained)
- No automatic linking of different stores to the same GTIN

## Non-Goals

1. **Atta Coupling**: Vareindeks is a standalone service that does not share database or user systems with Atta
2. **User Management**: No consumer accounts, emails, alerts or personalization features
3. **Inventory Tracking**: No local warehouse/branch inventory tracking
4. **Complex Search**: No advanced ranking or popularity-based search beyond exact GTIN/facets
5. **Data Processing**: No arbitrary code execution or data transformation beyond ingestion
6. **External Dependencies**: No mandatory Redis/Celery/OpenSearch/S3 infrastructure

## Validation

This product charter ensures alignment with the following invariants:
- I-001: Every published valid StoreProduct has exactly one non-null ProductIdentity FK
- I-002: Every ProductIdentity has stable UUID; canonical GTIN is nullable unique string
- I-003: `(source_id,external_key)` uniquely identifies one StoreProduct
- I-004: Two different Stores with same valid non-blocked GTIN resolve to one canonical ProductIdentity
- I-005: Source fields are not replaced by canonical normalized inference
- I-006: Blocked `(store_id,raw GTIN)` prevents automatic linking for that Store only
- I-007: Manual override persists through reimport and supersedes blocked behavior
- I-008: GTIN-less reimport with same key reuses both UUIDs
- I-009: Price and currency are Decimal/text on StoreProduct, nullable when unknown
- I-010: PriceHistory records only material changes atomically/idempotently
- I-012: `/v1/products` returns only ProductIdentity objects (including GTIN=NULL)
- I-013: Valid records automatically publish; individual bad rows isolated
- I-015: JobRun, batches, leases and price changes are crash-resilient
- I-016: Agent/run credential scopes are noninterchangeable
- I-017: Remote servers never send executable code; only allowlisted connectors
- I-018: No local warehouse/branch inventory or separate Offer/Listing model
- I-025: API `/v1` identifiers are stable and pagination/resumable