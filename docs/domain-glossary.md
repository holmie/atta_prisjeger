# Domain Glossary

This document defines the core domain models for Vareindeks and their relationships, as specified in the project requirements.

## Core Models

### Store
- **Definition**: Represents an independently priced seller with no branch inventory
- **Purpose**: Identifies distinct sellers in the system
- **Properties**: 
  - Unique identifier (UUID)
  - Name and other identifying information
  - Source-specific configuration
- **Key Characteristics**:
  - Each StoreProduct must reference a Store via foreign key
  - The Store foreign key in StoreProduct must identify the same Store as the DataSource foreign key

### DataSource
- **Definition**: Feed/connector that belongs to one Store
- **Purpose**: Defines how data is ingested from a specific source
- **Properties**:
  - Unique identifier (UUID)
  - Connection details and credentials
  - Store foreign key reference

### ProductIdentity
- **Definition**: Searchable UUID with nullable UNIQUE valid GTIN and variant metadata
- **Purpose**: Represents canonical products that can be linked across different stores. This is a canonical identity record, not a listing.
- **Properties**:
  - Stable UUID (primary key)
  - Canonical valid GTIN (nullable unique string, not primary key)
  - Variant metadata for product differentiation
- **Key Characteristics**:
  - UUID is stable and never changes
  - GTIN is nullable but unique when present
  - Each ProductIdentity represents a single canonical product

### StoreProduct
- **Definition**: Stable UUID with `(source_id, external_key)` unique constraint, linked to Store and ProductIdentity
- **Purpose**: Represents a specific product listing in a specific store with its price information
- **Properties**:
  - Stable UUID (primary key)
  - Source foreign key reference
  - External key from source system
  - ProductIdentity foreign key (NOT NULL)
  - Reported GTIN/SKU retained from source
  - Current Decimal price (nullable)
  - Currency (nullable text)
  - URL/status information
- **Key Characteristics**:
  - Each StoreProduct is unique by `(source_id, external_key)` combination
  - Always linked to exactly one ProductIdentity (never NULL)
  - Source fields including reported_gtin are never overwritten
  - PriceHistory references StoreProduct and records initial valid price + material changes only
  - Currency field is preserved from source and maintained alongside the price

### PriceHistory
- **Definition**: Append-only record of material price changes referencing StoreProduct UUID
- **Purpose**: Maintains complete timeline of price changes for each product in each store
- **Properties**:
  - References StoreProduct UUID
  - Records initial valid price and material changes only
  - No duplicate unchanged polls (refreshes freshness only)
  - Append-only storage
- **Key Characteristics**:
  - Atomic and idempotent recording of price events
  - Material changes only (unchanged polls refresh freshness)
  - Initial price event tracking

## Derived Models

### CurrentLowestPrice
- **Definition**: One-to-one cache of the current lowest price for a ProductIdentity
- **Purpose**: Provides fast access to the currently cheapest price across all stores for each canonical product
- **Properties**:
  - References Store UUID, StoreProduct UUID and store_name_snapshot
  - Updated on material price changes
  - Contains the actual derived lowest-price amount
  - Refreshed on merge, split, or relink of ProductIdentity that affects the cheapest StoreProduct
- **Key Characteristics**:
  - Derived/repairable cache
  - Maintains historical seller information without current JOINs
  - Updated not only on material price changes but also on merge, split, or relink operations that affect the cheapest StoreProduct

### LowestPriceHistory
- **Definition**: Event-based record of winner changes for lowest prices
- **Purpose**: Tracks historical changes in the cheapest price for each product across stores
- **Properties**:
  - Records material lowest-price, winner Store ID/name and availability-gap transitions
  - Carry forward known prices within covered intervals
  - Never pre-observation or known unavailable gaps
  - Contains the actual derived lowest-price amount (and currency/availability semantics where applicable)
  - Updated on material price changes and merge, split, or relink operations that affect the cheapest StoreProduct
- **Key Characteristics**:
  - Event-based tracking of price winner changes
  - Historical seller store_id, store_product_id, store_name_snapshot preserved
  - Bounded to 24 months plus pre-cutoff checkpoints
  - Stores the actual derived lowest-price amount (not just winner information)
  - Updated not only on material price changes but also on merge, split, or relink operations that affect the cheapest StoreProduct
  - **A relink can change an earlier winner or coverage gap while leaving today's winner unchanged**. However, when a relink operation affects the historical price records for affected ProductIdentities, the LowestPriceHistory must be recomputed to reflect the new state of the cheapest StoreProducts, even if the current winner remains the same.

## Source Truth vs Derived Data

### Source Truth
The following records represent source truth and are immutable or directly derived from source data:
- **StoreProduct**: Contains the original price information as reported by the source
- **PriceHistory**: Append-only record of actual price events (not estimates)
- **ProductIdentity**: Canonical product identity record with GTIN information

### Derived Data
The following records are computed/derived from source truth and can be recalculated:
- **CurrentLowestPrice**: Cache of current lowest price for each canonical product
- **LowestPriceHistory**: Historical record of lowest price winner changes including actual derived price amounts

## Lifecycle and Bounding

### Price History Bounding
- Retain 24 months of material price events plus required pre-cutoff checkpoints
- Logs/raw/ranking expire, incomplete feeds never bulk delete
- Raw feeds 14 days, verbose logs 90 days, raw ranking 7 days, aggregate ranking 90 days default
- Safe StoreProduct retirement after >=180 days and no dependent retained records

### Data Relationships
```
Store (1) -> (N) DataSource (1) -> (N) StoreProduct
ProductIdentity (1) -> (N) StoreProduct (1) -> (N) PriceHistory
ProductIdentity (1) -> (0..1) CurrentLowestPrice
ProductIdentity (1) -> (N) LowestPriceHistory

```

## Key Invariants

1. Every published valid StoreProduct has exactly one non-null ProductIdentity FK
2. Every ProductIdentity has stable UUID; canonical GTIN is a nullable unique string
3. `(source_id,external_key)` uniquely and idempotently identifies one StoreProduct
4. Source fields including reported_gtin never overwritten (auditable corrections)
5. Derived caches are repairable and historical seller information preserved without JOINs
6. History is bounded to 24 months plus pre-cutoff checkpoint
7. No automatic GTIN joining for a store if `(store_id, reported_gtin)` is blocked
8. Manual StoreProduct→ProductIdentity override wins over block/automatic GTIN behavior
9. **CurrentLowestPrice is computed per ProductIdentity (not StoreProduct)**
10. **LowestPriceHistory records contain the actual derived lowest-price amounts**

## Relink Impact on History

A relink operation can change an earlier winner or coverage gap while leaving today's winner unchanged. In such cases:
- Historical results for affected identities must be refreshed when their linked products change
- This ensures that if a previously non-cheapest product becomes the cheapest due to a relink, the history is updated appropriately
- **However, when a relink operation affects the historical price records for affected ProductIdentities, the LowestPriceHistory must be recomputed to reflect the new state of the cheapest StoreProducts, even if the current winner remains the same**

## Negative Test Case

### Proving Derived Lowest History Cannot Replace StoreProduct Price Truth

The derived `LowestPriceHistory` records are event-based winner changes that contain the actual derived lowest-price amounts computed from source data. They record:

- Which store had the lowest price at a given time
- When the winner changed
- Availability-gap transitions
- The actual derived lowest-price amount (and currency/availability semantics where applicable)

However, `LowestPriceHistory` cannot replace `StoreProduct` price truth because it only provides information about price winner changes and their associated actual prices, not the raw source price data itself. 

The key point is that while `LowestPriceHistory` contains the actual lowest-price amounts as derived from `StoreProduct` and `PriceHistory`, these derived records are still recomputable from source truth. This means:

1. **Source of truth remains**: The actual prices are stored in `StoreProduct` and `PriceHistory`
2. **Derived can be recomputed**: `LowestPriceHistory` records are repairable and can be recalculated from source data
3. **No data loss**: If you need the exact price from a specific store product, you must query `StoreProduct` or `PriceHistory`

**Note**: For an updated and consistent description of what `LowestPriceHistory` contains (including actual price amounts), please refer to [docs/negative-test.md](negative-test.md).

## Regression Test Scenario

To properly test that the implementation correctly handles all scenarios, we need a more comprehensive regression test:

### Planned Scenario: Price Change Impact on Lowest Price History
1. Create two StoreProducts with different stores and prices (e.g., 100.00 and 90.00)
2. Run derivation to establish baseline lowest price history
3. Change the losing seller's price from 100.00 to 95.00 
4. Verify that PriceHistory retains both amounts (100.00 and 95.00) 
5. Verify that LowestPriceHistory emits no new event because the winner is unchanged

This scenario tests the specific case mentioned in the feedback where an implementation might discard non-winning source prices, which would be a regression.