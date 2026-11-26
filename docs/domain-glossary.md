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

### DataSource
- **Definition**: Feed/connector that belongs to one Store
- **Purpose**: Defines how data is ingested from a specific source
- **Properties**:
  - Unique identifier (UUID)
  - Connection details and credentials
  - Store foreign key reference

### ProductIdentity
- **Definition**: Searchable UUID with nullable UNIQUE valid GTIN and variant metadata
- **Purpose**: Represents canonical products that can be linked across different stores
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
  - URL/status information
- **Key Characteristics**:
  - Each StoreProduct is unique by `(source_id, external_key)` combination
  - Always linked to exactly one ProductIdentity (never NULL)
  - Source fields including reported_gtin are never overwritten
  - PriceHistory references StoreProduct and records initial valid price + material changes only

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
- **Definition**: One-to-one cache of the current lowest price for a StoreProduct
- **Purpose**: Provides fast access to the currently cheapest price for each product
- **Properties**:
  - References Store UUID, StoreProduct UUID and store_name_snapshot
  - Updated on material price changes
- **Key Characteristics**:
  - Derived/repairable cache
  - Maintains historical seller information without current JOINs

### LowestPriceHistory
- **Definition**: Event-based record of winner changes for lowest prices
- **Purpose**: Tracks historical changes in the cheapest price for each product across stores
- **Properties**:
  - Records material lowest-price, winner Store ID/name and availability-gap transitions
  - Carry forward known prices within covered intervals
  - Never pre-observation or known unavailable gaps
- **Key Characteristics**:
  - Event-based tracking of price winner changes
  - Historical seller store_id, store_product_id, store_name_snapshot preserved
  - Bounded to 24 months plus pre-cutoff checkpoints

## Source Truth vs Derived Data

### Source Truth
The following records represent source truth and are immutable or directly derived from source data:
- **StoreProduct**: Contains the original price information as reported by the source
- **PriceHistory**: Append-only record of actual price events (not estimates)
- **ProductIdentity**: Canonical product identity with GTIN information

### Derived Data
The following records are computed/derived from source truth and can be recalculated:
- **CurrentLowestPrice**: Cache of current lowest price for each product
- **LowestPriceHistory**: Historical record of lowest price winner changes

## Lifecycle and Bounding

### Price History Bounding
- Retain 24 months of material price events plus required pre-cutoff checkpoints
- Logs/raw/ranking expire, incomplete feeds never bulk delete
- Raw feeds 14 days, verbose logs 90 days, raw ranking 7 days, aggregate ranking 90 days default
- Safe StoreProduct retirement after >=180 days and no dependent retained records

### Data Relationships
```
Store
  └── DataSource (1:N)
        └── StoreProduct (1:N)
              └── PriceHistory (1:N)
                    └── CurrentLowestPrice (1:1)
                    └── LowestPriceHistory (1:N)
              └── ProductIdentity (N:1)
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

## Negative Test Case

### Proving Derived Lowest History Cannot Replace StoreProduct Price Truth

The derived `LowestPriceHistory` records are event-based winner changes and do not contain the actual price values that were recorded in `StoreProduct` or `PriceHistory`. They only track:
- Which store had the lowest price at a given time
- When the winner changed
- Availability-gap transitions

However, they do NOT preserve the actual price amounts from the source `StoreProduct` records. This is by design because:

1. **Source of truth remains**: The actual prices are stored in `StoreProduct` and `PriceHistory`
2. **Derived only tracks winners**: `LowestPriceHistory` tracks who was the cheapest, not what the exact price was
3. **No price data loss**: If you need to know the actual price that was lowest at a given time, you must query the `PriceHistory` for that specific store product

Therefore, `LowestPriceHistory` cannot replace `StoreProduct` price truth because it only provides information about price winner changes, not the actual prices themselves.