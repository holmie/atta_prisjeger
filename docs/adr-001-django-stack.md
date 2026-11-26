# ADR-001: Django/DRF/PostgreSQL Control Plane

## 1. Context

Vareindeks needs a robust and lightweight control plane to manage its core functionality including data ingestion, job scheduling, and API services.

## 2. Options

1. **Django/DRF/PostgreSQL 16**: Use Django as the web framework, DRF for REST APIs, and PostgreSQL 16 as the database
2. **Flask/SQLAlchemy/PostgreSQL**: Use Flask as the web framework, SQLAlchemy as ORM, and PostgreSQL as the database  
3. **FastAPI/SQLAlchemy/PostgreSQL**: Use FastAPI as the web framework, SQLAlchemy as ORM, and PostgreSQL as the database

## 3. Decision

We choose Django/DRF/PostgreSQL 16 for the control plane because:
- It's a proven stack with extensive documentation and community support
- DRF provides robust REST API capabilities out of the box
- Django's admin interface is useful for operational tasks
- The team has existing expertise with this stack

## 4. Tradeoffs

**Pros:**
- Mature ecosystem with extensive documentation
- Built-in admin interface for operations
- Strong ORM support
- Large community and third-party packages

**Cons:**
- Slightly heavier than alternatives like Flask or FastAPI
- Less flexible in terms of routing compared to more minimal frameworks

## 5. Tests

1. Unit tests for all models, views, and serializers
2. Integration tests for API endpoints
3. Negative test: Ensure no undocumented changes to contracts
4. Test that all data access follows the established patterns

## 6. Supersession

This supersedes any previous architectural decisions about the control plane technology stack.

## 7. Authority Rule

This decision is made by the core development team and approved by project lead.

## 8. Validation

We validate this decision through:
- Successful implementation of core features
- Performance testing
- Code reviews
- Operational feedback from system monitoring

## 9. Related Documents

- D-001: Separate Django/DRF/PostgreSQL control plane (project-memory/01_DECISIONS.md)
- I-001: Every published valid StoreProduct has exactly one non-null ProductIdentity FK