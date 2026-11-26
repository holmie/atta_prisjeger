# Vareindeks Agents

This document defines the architecture boundaries, PR discipline, and operational constraints for the Vareindeks project.

## Agent Architecture

Vareindeks operates as a standalone Django/DRF/PostgreSQL 16 control plane with outbound-only Python agents. All agents run from low-powered Linux environments (WSL/VPS) without inbound ports.

## PR Discipline

- Single task per PR; split before implementing if work exceeds single-task scope
- No cross-repo imports or arbitrary remote execution
- All changes must be reviewed and merged by human operator
- Never merge, claim release readiness, or run live scrapers without explicit permission/rights

## Source Rights & Safety

- Agent packages use outbound HTTPS only
- Installed allowlisted connectors; server may not distribute executable code
- Remote servers never send arbitrary executable Python code
- No routine human approval queue for valid records
- Explicit 403/429 respects access policy
- Live source rights and redistribution scope must be verified independently

## Operational Constraints

- No Atta coupling or shared database
- No parallel Offer/Listing/search-pseudo-product model
- No mandatory manual import approval
- No local inventory-location tracking
- No unrelated refactors, arbitrary code dispatch or unapproved live scrapes
- Preserve existing public contracts, Decimal money, source rights and scoped auth

## Documentation

This project uses the following canonical documents:
- `project-memory/00_CONTEXT.md` - Project context
- `project-memory/01_DECISIONS.md` - Design decisions
- `project-memory/02_INVARIANTS.md` - Hard invariants
- `project-memory/03_CONTRACTS.md` - API contracts
- `project-memory/04_CURRENT_STATE.md` - Current state documentation
- `contracts/*.md` - Detailed contract specifications

## Negative Test

This repository contains no references to Atta codebase, confirming independence from the Atta system.