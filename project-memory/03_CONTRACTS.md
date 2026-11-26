# 03 — API contracts

This document defines the API contracts for Vareindeks.

## `/v1/products`

- Returns ProductIdentity objects (including those with NULL GTIN)
- Searchable by GTIN, title and other facets
- Exact GTIN/filters outrank popularity
- No routine human approval queue
- No personalization or user-specific data

## `/v1/prices`

- Returns price information for StoreProducts
- Includes current price, historical price changes, and lowest price history
- Derived from linked StoreProducts

## `/v1/stores`

- Returns information about stores
- Each store is an independently priced seller
- No physical inventory sites

## `/v1/data-sources`

- Returns information about data sources
- Feed/connector belongs to one Store

## `/v1/store-products`

- Returns information about store products
- Includes reported GTIN, SKU, title, URL and status
- Links to ProductIdentity

## Authentication

- API keys required for read access
- Operator sessions with administrative privileges
- Agent/run credential, push source credential with noninterchangeable scopes

## Error Handling

- Standard HTTP status codes
- Error messages in JSON format
- No sensitive information exposed in responses