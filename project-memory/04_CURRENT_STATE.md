# 04 — Current State

## Project Status

This is the initial setup of the Vareindeks repository. The project is in its early stages with the foundational architecture and documentation in place.

## Architecture

- Django/DRF/PostgreSQL 16 control plane
- Outbound-only Python agents
- Source-scoped push ingest
- No inbound ports (runs on low-powered Linux environments like WSL/VPS)

## Implemented Components

- AGENTS.md - Agent architecture and operational constraints
- Project memory files:
  - 00_CONTEXT.md - Project context and model definitions
  - 01_DECISIONS.md - Design decisions and ADRs
  - 02_INVARIANTS.md - Hard invariants requiring tests
  - 03_CONTRACTS.md - API contracts

## Next Steps

- Implementation of core models (Store, DataSource, ProductIdentity, StoreProduct, PriceHistory)
- Agent implementation for data ingestion
- API endpoint development
- Testing and validation of invariants