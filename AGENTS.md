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
- `contracts/lowest-price.md` - Lowest price calculation contract
- `docs/adr-template.md` - Architecture Decision Record template
- `docs/domain-glossary.md` - Domain model definitions and relationships

## OpenCode/Codex Workflow

This repository is managed by OpenCode/Codex agents with the following workflow:

1. Tasks are delivered via external/local plan, not committed to this repo
2. Read relevant in-repo memory/contracts before implementation:
   - `project-memory/00_CONTEXT.md`
   - `project-memory/01_DECISIONS.md`
   - `project-memory/02_INVARIANTS.md`
   - `project-memory/03_CONTRACTS.md`
   - `contracts/lowest-price.md`
3. Implement only the assigned task
4. Run focused tests and checks:
   - Run relevant unit tests: `python manage.py test` (only after Django project with manage.py has been bootstrapped)
   - Verify against invariants in `project-memory/02_INVARIANTS.md`
5. Report evidence of testing in PR description
6. Create PR with title starting `PJ-XXXX: <title>` (where XXXX is the current assigned task ID)
7. Do not merge or mark DONE until human operator reviews and merges

## Negative Test

This repository currently contains no application code to check for cross-repo dependencies. When such files exist, a future/repeatable dependency inspection (e.g., imports, dependency manifests, environment/database configuration) will be used to verify independence from the Atta system. Atta may be documented as an independent consumer.