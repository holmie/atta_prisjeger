# Lowest Price Contract

## Overview

This document defines the contract for computing and storing the lowest price information for products in Vareindeks.

## Derived Models

### CurrentLowestPrice
- One-to-one cache of the current lowest price
- References Store UUID, StoreProduct UUID and store_name_snapshot
- Updated on material price changes or availability transitions that can change the current result

### LowestPriceHistory
- Event-based record of winner changes
- Records material lowest-price, winner Store ID/name 
- Availability-gap transitions
- Carry forward known prices within covered intervals
- Never pre-observation or known unavailable gaps
- Updated on availability transitions that change the winner or open/closed coverage gaps, or when price history events change the result

## Requirements

- History is bounded to 24 months plus pre-cutoff checkpoint
- Records material lowest-price, winner Store ID/name and availability-gap transitions
- Carry forward known prices within covered intervals, never pre-observation or known unavailable gaps
- Historical seller store_id, store_name_snapshot preserved without a current Store-name JOIN

## Data Flow

- Derived from StoreProduct price history, including preserved price observations and supported availability and coverage evidence for currently linked offers
- Computed for currently linked StoreProducts
- With coverage gaps (simple transparent v1 query semantics)
- Raw history never rewritten

## Refresh Rules

Both CurrentLowestPrice and LowestPriceHistory projections are refreshed according to this single authoritative rule:

**When any change affects the set of currently linked StoreProducts for a ProductIdentity, refresh both projections for that identity.**

This includes:
- Material price changes in currently linked StoreProducts
- Availability/eligibility transitions that can change the current or historical result (including when a cached winner becomes unavailable at an unchanged price)
- Identity merge/split/relink operations affecting currently linked StoreProducts
- Any change that affects which StoreProducts are considered "currently linked" to a ProductIdentity

This rule ensures that both projections are always consistent with each other and with the current state of linked StoreProducts, including cases where:
1. A StoreProduct is relinked from one ProductIdentity to another (involving both identities)
2. A previously winning StoreProduct becomes unavailable at an unchanged price
3. Any change that affects the membership of currently linked StoreProducts for any identity

## Validation Requirements

- When a cached winner becomes unavailable at an unchanged price, the refresh must yield a new current winner or no eligible result
- Rebuilding history must reproduce the winner or gap transition without a new PriceHistory event
- Historical rebuilds must not modify raw per-seller prices or PriceHistory records