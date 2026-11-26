# 03 — API contracts

This document defines the API contracts for Vareindeks.

## `/v1/products`

- Returns ProductIdentity objects (including those with NULL GTIN)
- Searchable by GTIN, title and other facets
- Exact GTIN/filters outrank popularity
- No routine human approval queue
- No personalization or user-specific data

## `/v1/store-products`

- Returns information about store products
- Includes reported GTIN, SKU, title, URL and status
- Links to ProductIdentity

## `/v1/price-history`

- Returns price history for a given StoreProduct
- Append-only material changes referencing StoreProduct UUID
- Initial price event, no duplicate unchanged polls

## Authentication

- API keys required for read access
- Operator sessions with administrative privileges
- Agent/run credential, push source credential with noninterchangeable scopes

## Error Handling

- Standard HTTP status codes
- Error messages in JSON format
- No sensitive information exposed in responses