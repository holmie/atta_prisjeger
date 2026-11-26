# Lowest Price Contract

## Overview

This document defines the contract for computing and storing the lowest price information for products in Vareindeks.

## Derived Models

### CurrentLowestPrice
- One-to-one cache of the current lowest price
- References Store UUID, StoreProduct UUID and store_name_snapshot
- Updated on material price changes

### LowestPriceHistory
- Event-based record of winner changes
- Records material lowest-price, winner Store ID/name 
- Availability-gap transitions
- Carry forward known prices within covered intervals
- Never pre-observation or known unavailable gaps

## Requirements

- History is bounded to 24 months plus pre-cutoff checkpoint
- Records material lowest-price, winner Store ID/name and availability-gap transitions
- Carry forward known prices within covered intervals, never pre-observation or known unavailable gaps
- Historical seller store_id, store_product_id, store_name_snapshot preserved without a current Store-name JOIN

## Data Flow

- Derived from StoreProduct price history
- Computed for currently linked StoreProducts
- With coverage gaps (simple transparent v1 query semantics)
- Raw history never rewritten