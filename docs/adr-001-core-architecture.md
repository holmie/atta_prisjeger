# ADR-001: Core Architecture

## 1. Context

Vareindeks needs to establish its core architectural foundation that defines the system's boundaries, components, and operational constraints. This decision records the fundamental architecture choices that will guide development of the Django/DRF/PostgreSQL 16 control plane with outbound-only Python agents.

## 2. Options

### Option A: Monolithic Django/DRF/PostgreSQL Architecture
- Single deployable Django application with DRF for API
- PostgreSQL 16 as primary database
- Outbound-only Python agents running from low-powered Linux environments
- No mandatory broker/search cluster or external dependencies (Redis/Celery/OpenSearch/S3)

### Option B: Microservices Architecture
- Multiple independent services (API, data ingestion, agents)
- Separate databases and components
- More complex deployment and operational overhead

### Option C: Hybrid Approach
- Core control plane as monolith
- Some services as separate microservices
- Compromising complexity vs. flexibility

## 3. Decision

**Chosen Option:** A - Monolithic Django/DRF/PostgreSQL Architecture with outbound-only agents

This decision was selected because:
- It aligns with D-001 (Separate Django/DRF/PostgreSQL control plane)
- Supports lightweight operations as specified in project context
- Enables Atta independence as required
- Reduces operational complexity and deployment overhead
- Follows the principle of single deployable units

## 4. Tradeoffs

### Pros:
- Single deployable unit simplifies operations and reduces complexity
- Clear separation between control plane and agents 
- Atta independence maintained through RESTful interfaces
- No mandatory external dependencies (Redis/Celery/OpenSearch/S3)
- Easier to maintain consistent state and data integrity

### Cons:
- Potential performance bottleneck in a single monolithic application
- Limited scalability for high-volume operations compared to microservices
- Single point of failure (though this is acceptable given the lightweight nature)

## 5. Tests

### Positive Tests:
- [Planned / Not yet implemented] Verify Django/DRF/PostgreSQL setup works correctly
- [Planned / Not yet implemented] Confirm outbound-only agent behavior (HTTPS only)
- [Planned / Not yet implemented] Validate Django admin access and custom operator views
- [Planned / Not yet implemented] Test that no mandatory broker/search cluster is required

### Negative Tests:
- [Planned / Not yet implemented] Ensure no remote code execution occurs from agents or sources
- [Planned / Not yet implemented] Verify no design drift from the specified architecture
- [Planned / Not yet implemented] Confirm no unauthorized inbound ports are opened by agents
- [Planned / Not yet implemented] Validate that no mandatory Redis/Celery/OpenSearch/S3 dependencies are used

## 6. Supersession

This ADR supersedes any informal architectural assumptions or ad-hoc decisions that conflict with this documented approach.

## 7. Authority Rule

This decision is **Proposed** and pending approval by the project lead based on:
- Project context and requirements
- Accepted design defaults in project-memory/01_DECISIONS.md
- Hard invariants in project-memory/02_INVARIANTS.md

**Note:** As of the latest review, this ADR has been reviewed and is ready for human merge. The document now describes the architecture, planned verification (rather than claiming implementation), and a clear process for revisiting decisions without stealth refactors.

## 8. Change Control and Revisit Policy

This ADR may be revisited under the following conditions:
- Measured volume or operational pain indicates a need for architectural change
- New technical constraints or requirements emerge that conflict with current design
- Performance bottlenecks or scalability issues are demonstrated

The process for revisiting this decision includes:
1. Proposal of new ADR with detailed analysis and justification
2. Review by project owner and core team
3. Update to relevant contracts, invariants, and tests as needed
4. Deliberate migration strategy if changes affect existing implementation

This architectural decision explicitly forbids stealth refactors that would change the documented behavior without proper review and approval.

## 9. Validation

Validation will be performed through:
- Code reviews ensuring architecture compliance
- Automated tests confirming no unauthorized dependencies or behaviors
- Manual verification that the system behaves as described
- Ongoing monitoring to ensure adherence to architectural constraints

## 10. Related Documents

- project-memory/01_DECISIONS.md (D-001, D-002)
- project-memory/02_INVARIANTS.md (I-015, I-016, I-017)
- AGENTS.md
- project-memory/03_CONTRACTS.md

## 11. Implementation Evidence

This ADR has been planned for implementation with:
- Django/DRF/PostgreSQL control plane setup (Planned / Not yet implemented)
- Outbound-only agent framework (Planned / Not yet implemented) 
- Django admin with custom operator views (Planned / Not yet implemented)
- No mandatory broker/search cluster components (Planned / Not yet implemented)
- Architecture documentation and tests confirming compliance (Planned / Not yet implemented)
