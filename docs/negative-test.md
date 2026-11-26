# Planned Negative Tests: Source Price Truth and Derived Lowest Price

**Status: specification only.** This repository does not yet contain executable application tests for these scenarios. Nothing below has been run or passed. Implement these as automated tests once ingestion and lowest-price derivation exist.

These cases exercise `project-memory/02_INVARIANTS.md` (especially I-009, I-010, I-011, I-020, I-021 and I-027), D-017, and `contracts/lowest-price.md`.

## 1. Losing seller's price change must survive ingestion

**Failure being excluded:** an implementation saves only the cheapest seller's prices, making `LowestPriceHistory` look correct while silently discarding another StoreProduct's source prices.

**Fixture and actions (through the normal ingestion entry point):**

1. Create two distinct Stores, each with its own DataSource. Ingest one StoreProduct per source. Record their **distinct StoreProduct UUIDs** as `sp_a` and `sp_b`; never substitute Store IDs for StoreProduct IDs.
2. Link both StoreProducts to the **same ProductIdentity**. Use the same known currency (e.g. NOK), simultaneously available offers, and overlapping known coverage.
3. At `t0`, ingest `sp_a = 100.00 NOK` and `sp_b = 90.00 NOK`. Run the normal minimum derivation. `sp_b` is the winner.
4. At `t1 > t0`, ingest a material price change for the **losing** `sp_a` from `100.00` to `95.00 NOK`, without changing eligibility or availability. Run derivation again.

**Required assertions:**

- `sp_a.current_price = 95.00` with currency NOK.
- `PriceHistory` for **`sp_a`** contains its initial `100.00` event and later `95.00` event, keyed to `sp_a`; `sp_b` retains its own `90.00` price event. No unchanged-poll duplicates.
- `CurrentLowestPrice` for the ProductIdentity still has **90.00 NOK**, winner `sp_b`, the correct Store UUID and copied `store_name_snapshot`.
- `LowestPriceHistory` contains the derived **90.00 NOK** minimum and winner; **the losing seller's 100-to-95 change alone produces no new lowest-price event**, because neither amount nor winner nor coverage changed.
- Corrupting or deleting the *derived* projection and rebuilding it from preserved source observations restores the same result; changing a projection does not change `sp_a` or either StoreProduct's `PriceHistory`.

This is the negative test: **a matching derived winner is insufficient** unless the losing seller's observations are demonstrably preserved by normal ingestion.

## 2. Relink may rewrite the historical minimum without changing today's winner

**Failure being excluded:** code that skips historical recomputation whenever the *current* winning seller remains unchanged.

Use known, overlapping availability and the same currency for all examples. Ingest the observations and derive history for two canonical identities `A` and `B`:

| StoreProduct | Initially linked identity | At t0 | At t1 (current) |
| --- | --- | --- | --- |
| `sp_x` | A | 100.00 | 80.00 |
| `sp_y` | A | 90.00 | 85.00 |
| `sp_z` | B | 120.00 | 110.00 |

Before relink, the expected minimum for A is `sp_y / 90.00` at t0 and `sp_x / 80.00` at t1. For B it is `sp_z / 120.00` at t0 and `sp_z / 110.00` at t1.

Now **relink `sp_y` from A to B** through the supported audited identity-correction path. Do not change prices or availability.

**Required assertions:**

- A's **current** winner remains `sp_x / 80.00`, but A's **historical** t0 minimum becomes `sp_x / 100.00` rather than `sp_y / 90.00`.
- B's current winner becomes `sp_y / 85.00`; its historical t0 minimum becomes `sp_y / 90.00` rather than `sp_z / 120.00`.
- The retained `LowestPriceHistory` timelines of **both** identities are recomputed using their **currently linked** StoreProducts. Earlier derived events may be inserted, replaced, or removed; there is **no rule forbidding historical changes when today's winner is unchanged**.
- All three StoreProduct UUIDs and their source `PriceHistory` events remain unchanged. No source price event is introduced solely for the relink.

This is a **historical rebuild**, not an invented source-price observation at relink time.

## 3. Availability-only transition affects the derived result, not PriceHistory

**Failure being excluded:** a cached winner remains the cheapest even after going unavailable because its numeric price did not change.

1. Ingest two StoreProducts linked to one ProductIdentity, with known comparable currency and coverage: `sp_a = 90.00 NOK` and `sp_b = 100.00 NOK`, both initially available.
2. Ingest a supported **availability-only** transition making `sp_a` unavailable; do not send a material price change.
3. Recompute affected current and historical minimum results. If `sp_b` remains eligible, it becomes the 100.00 NOK winner. If no offer remains eligible, represent the coverage gap rather than inventing a lowest price.
4. Assert that `LowestPriceHistory` records the relevant winner/gap transition and `CurrentLowestPrice` no longer presents `sp_a` as eligible.
5. Assert that **neither StoreProduct gets a new PriceHistory price event from availability alone** (I-010).

The system must use available observation/coverage evidence; no historical availability is fabricated where the source does not establish it.

---

**Acceptance evidence required later:** executable test names, the exact command run, and actual results. These written scenarios alone do not satisfy a passing-test claim.
