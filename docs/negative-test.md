# Negative Test: Derived Lowest History Cannot Replace StoreProduct Price Truth

## Test Case

This test demonstrates that `LowestPriceHistory` records are event-based winner changes and cannot replace `StoreProduct` price truth.

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
- Does NOT contain the actual price value (90.00) from the StoreProduct

### Verification

To verify that `LowestPriceHistory` cannot replace `StoreProduct` price truth:
1. Query the `StoreProduct` with ID 2 to get actual price of 90.00
2. Compare with `LowestPriceHistory` which only shows winner information
3. Confirm that `LowestPriceHistory` lacks the exact price amount

### Conclusion

This demonstrates that:
- Source truth remains in `StoreProduct` and `PriceHistory`
- `LowestPriceHistory` is derived and serves a different purpose (tracking winners)
- The actual prices must be retrieved from source records, not derived records