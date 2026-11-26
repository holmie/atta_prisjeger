# Negative Test: Derived Lowest History Cannot Replace StoreProduct Price Truth

## Test Case

This test demonstrates that `LowestPriceHistory` records are event-based winner changes that do contain actual price values, and cannot replace `StoreProduct` price truth.

### Setup
1. Create a `StoreProduct` with initial price of 100.00
2. Create a `PriceHistory` entry for this price
3. Create another `StoreProduct` with the same product but different store, price 90.00
4. Create a `PriceHistory` entry for this price
5. Create `LowestPriceHistory` record tracking that second store was cheapest

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

### Conclusion

This demonstrates that:
- Source truth remains in `StoreProduct` and `PriceHistory`
- `LowestPriceHistory` is derived and serves a different purpose (tracking winners and their actual prices)
- The actual prices are preserved in both source and derived records
- However, the derived records are still recomputable from source data

## Regression Test Scenario

### Planned Scenario: Price Change Impact on Lowest Price History
1. Create two StoreProducts with different stores and prices (e.g., 100.00 and 90.00)
2. Run derivation to establish baseline lowest price history
3. Change the losing seller's price from 100.00 to 95.00 
4. Verify that PriceHistory retains both amounts (100.00 and 95.00) 
5. Verify that LowestPriceHistory emits no new event because the winner is unchanged

This scenario tests the specific case mentioned in the feedback where an implementation might discard non-winning source prices, which would be a regression.

## Correction Based on Feedback

The original documentation incorrectly stated that `LowestPriceHistory` did not contain price values. This has been corrected to reflect that:
1. `LowestPriceHistory` records contain the actual derived lowest-price amounts
2. These amounts are computed from `StoreProduct` and `PriceHistory` data
3. The derived records are still repairable/recomputable from source data, but now include the actual price values