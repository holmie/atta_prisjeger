# 04 — Current state

## Overview

This document provides a snapshot of the current state of the Vareindeks project. It is updated with each significant change to the project.

## Project Status

The project is in its initial phase, with the following key components:

- Django/DRF/PostgreSQL 16 control plane
- Outbound-only Python agents
- Source-scoped push ingest
- No Atta coupling or shared database
- No parallel Offer/Listing/search-pseudo-product model
- No mandatory manual import approval
- No local inventory-location tracking

## Design Intentions (as defined in project-memory/01_DECISIONS.md)

- ProductIdentity objects are created for every valid StoreProduct
- StoreProduct UUID, source external key and mandatory ProductIdentity FK ensure reimport idempotency
- PriceHistory records append-only material changes referencing StoreProduct UUID
- Source fields including reported_gtin never overwritten
- BlockedStoreGTIN prevents automatic join for a specific store only
- Manual StoreProductIdentityOverride wins over block/automatic GTIN

## Implemented Functionality

- Basic project structure and repository setup
- Product charter documentation
- Core project memory files (context, decisions, invariants, contracts, current state)

## Next Steps

1. Implement core models as defined in project-memory/01_DECISIONS.md and project-memory/02_INVARIANTS.md
2. Develop data ingestion pipelines
3. Implement API endpoints
4. Add tests for all components
5. Set up monitoring and logging

## Known Issues

- No current implementation of the full feature set
- Missing data validation and error handling
- No user authentication or authorization system