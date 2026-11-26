# Negative Test: Derived Lowest History Cannot Replace StoreProduct Price Truth

## Test Case

This test demonstrates that `LowestPriceHistory` records are event-based winner changes that do contain actual price values, and cannot replace `StoreProduct` price truth.

### Setup
1. Create a `StoreProduct` with initial price of 100.00
2. Create a `PriceHistory` entry for this price
3. Create another `StoreProduct` with the same product but different store, price 90.00
4. Create a `PriceHistory` entry for this price
5. Run derivation to establish baseline lowest price history (store with ID 2 is cheapest)

### Expected Behavior

The `LowestPriceHistory` record:
- Shows store with ID 2 had the lowest price 
- Records the time of change
- Preserves store_id, store_product_id, and store_name_snapshot
- **DOES contain the actual price value (90.00) from the StoreProduct** as part of the derived lowest-price amount

### Verification

To verify that `LowestPriceHistory` cannot replace `StoreProduct` price truth:
1. Query the `StoreProduct` with ID 2 to get actual price of 90.00
2. Compare with `LowestPriceHistory` which now shows the exact price amount (90.00)
3. Confirm that `LowestPriceHistory` does contain the exact price amount

## Updated Test Scenario for Feedback Compliance

The following scenario better demonstrates that losing prices are preserved in PriceHistory:

1. Create two StoreProducts with different stores and prices (e.g., 100.00 and 90.00) 
2. Link both sellers to the same ProductIdentity
3. Run derivation to establish baseline lowest price history (store with ID 2 is cheapest)
4. Change the losing seller's price from 100.00 to 95.00 
5. Verify that PriceHistory retains both amounts (100.00 and 95.00) 
6. Verify that LowestPriceHistory emits no new event because the winner is unchanged
7. **Verify that the losing StoreProduct still shows 95.00 in its current price**
8. **Verify that the losing StoreProduct's PriceHistory still contains both 100.00 and 95.00 entries**

## Correction Based on Feedback

The original documentation incorrectly stated that `LowestPriceHistory` did not contain price values. This has been corrected to reflect that:

1. `LowestPriceHistory` records contain the actual derived lowest-price amounts
2. These amounts are computed from `StoreProduct` and `PriceHistory` data
3. The derived records are still repairable/recomputable from source data, but now include the actual price values

### Additional Clarification on Relink Impact

A relink operation can change an earlier winner or coverage gap while leaving today's winner unchanged. In such cases:

1. Historical results for affected identities must be refreshed when their linked products change
2. This ensures that if a previously non-cheapest product becomes the cheapest due to a relink, the history is updated appropriately
3. If the current winner remains unchanged, no new events are added to LowestPriceHistory
4. However, under D-017 and I-021, recomputing history from the currently linked StoreProducts can add, replace, or remove earlier price, winner, and availability-gap events. This means that when a relink operation affects the historical price records for affected ProductIdentities, the `LowestPriceHistory` must be recomputed to reflect the new state of the cheapest StoreProducts, even if the current winner remains the same.

## Regression Test Scenario

### Planned Scenario: Price Change Impact on Lowest Price History
1. Create two StoreProducts with different stores and prices (e.g., 100.00 and 90.00)
2. Link both sellers to the same ProductIdentity
3. Run derivation to establish baseline lowest price history (store with ID 2 is cheapest)
4. Change the losing seller's price from 100.00 to 95.00 
5. Verify that PriceHistory retains both amounts (100.00 and 95.00) 
6. Verify that LowestPriceHistory emits no new event because the winner is unchanged
7. Verify that the losing StoreProduct still shows 95.00 in its current price
8. Verify that the losing StoreProduct's PriceHistory still contains both 100.00 and 95.00 entries

This scenario tests the specific case mentioned in the feedback where an implementation might discard non-winning source prices, which would be a regression.

### Updated Test Scenario with Proper Ingestion Path

To ensure proper testing of the ingestion path and prevent regressions:

1. **Create two StoreProducts with different stores and prices** (e.g., 100.00 and 90.00) using the normal ingestion process
2. **Link both sellers to the same ProductIdentity**
3. **Run derivation to establish baseline lowest price history** (store with ID 2 is cheapest)
4. **Update the losing seller's price through normal ingestion** from 100.00 to 95.00 
5. **Verify that PriceHistory retains both amounts** (100.00 and 95.00) using direct queries on the PriceHistory records
6. **Verify that LowestPriceHistory emits no new event** because the winner is unchanged
7. **Verify that the losing StoreProduct still shows 95.00 in its current price**
8. **Verify that the losing StoreProduct's PriceHistory still contains both 100.00 and 95.00 entries**

This updated scenario ensures:
- All data enters through the normal ingestion path
- The test verifies that losing prices are properly preserved in PriceHistory
- It demonstrates that the system correctly handles both current and historical price records
- It prevents the regression where non-winning prices might be discarded