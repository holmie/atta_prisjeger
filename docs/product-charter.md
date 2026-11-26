# Product Charter

## Purpose

Vareindeks is a standalone, lightweight product/price API and jobs control plane designed to provide price data for the Atta system. It operates as a separate Django/DRF/PostgreSQL 16 control plane with outbound-only Python agents and source-scoped push ingest. The system is designed to be independent of Atta's user system, consumer login, personal price alerts or bonus engine.

## Actors

- **Atta System**: The primary consumer of the Vareindeks API
- **Data Sources**: Outbound-only connectors that provide product data
- **Operators**: Human administrators who manage the system
- **Agents**: Automated processes that fetch data from external sources

## V1 Scope

- ProductIdentity objects are created for every valid StoreProduct
- StoreProduct UUID, source external key and mandatory ProductIdentity FK ensure reimport idempotency
- PriceHistory records append-only material changes referencing StoreProduct UUID
- Source fields including reported_gtin never overwritten
- BlockedStoreGTIN prevents automatic join for a specific store only
- Manual StoreProductIdentityOverride wins over block/automatic GTIN

## Exclusions

This product charter explicitly excludes:

1. Consumer accounts, emails, alerts and Atta reward computation
2. Atta coupling or shared database
3. Parallel Offer/Listing/search-pseudo-product model
4. NULL ProductIdentity FK
5. Mandatory manual import approval
6. Local inventory-location tracking
7. Unrelated refactors, arbitrary code dispatch or unapproved live scrapes
8. Any changes to public contracts, Decimal money, source rights and scoped auth

## Negative Test

The exclusion list has been checked against the requirements and confirmed to be consistent with the design decisions and invariants.