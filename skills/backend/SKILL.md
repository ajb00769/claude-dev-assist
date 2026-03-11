---
name: backend
description: Backend development scaffolding patterns. Use when user says /backend, needs to scaffold endpoints, services, models, or wants backend best practices. Auto-detects the project's tech stack and provides idiomatic patterns.
---

# Backend Scaffolding Patterns

Quick reference for backend development patterns. Auto-detect the stack from project files before scaffolding.

## Stack Detection

Before scaffolding, identify the stack:

| File | Stack |
|------|-------|
| `pyproject.toml` / `requirements.txt` + FastAPI | Python / FastAPI |
| `pyproject.toml` / `requirements.txt` + Django | Python / Django |
| `pyproject.toml` / `requirements.txt` + Flask | Python / Flask |
| `package.json` + Express/Fastify/Nest | Node.js |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pom.xml` / `build.gradle` + Spring | Java / Spring |
| `Gemfile` + Rails | Ruby on Rails |
| `mix.exs` + Phoenix | Elixir / Phoenix |

Then follow existing patterns in the codebase. Read at least 2-3 existing files of the same type before scaffolding new ones.

## Scaffolding Decision Tree

**Adding new functionality?**
- New API endpoint → See "New Endpoint"
- New database table → See "New Model"
- New business logic → See "New Service"
- New validation schema → See "New Schema"

## New Endpoint

1. Check existing route files for the project's pattern (decorator style, router registration, middleware usage)
2. Follow the established convention for:
   - Route registration method
   - Authentication/authorization middleware
   - Request/response serialization
   - Error handling pattern
   - Dependency injection style

**Universal principles**:
- Thin route handlers — delegate business logic to services
- Authentication on protected routes
- Rate limiting on expensive operations (AI, bulk processing)
- Input validation via schema/type system
- Consistent error response format

## New Model

1. Check existing models for the project's ORM/data layer pattern
2. Follow the established convention for:
   - Column/field types
   - Relationships/associations
   - Naming conventions (table names, column names)
   - Timestamps, soft delete, audit fields
   - Index definitions

**Universal principles**:
- Foreign keys with appropriate cascade behavior
- Indexes on frequently queried columns
- Nullable fields explicitly declared
- Type-safe field definitions

## New Service

1. Check existing service files for the project's pattern
2. Follow the established convention for:
   - Function/method signatures
   - Database session/connection handling
   - Transaction boundaries
   - Error handling and logging
   - Return types

**Universal principles**:
- Single responsibility — one service per domain concept
- Eager loading for relationships (prevent N+1)
- Batch operations for bulk writes
- Transaction blocks for atomic operations
- Ownership/authorization checks in queries

## New Schema

1. Check existing schema/DTO files for the project's validation pattern
2. Follow the established convention for:
   - Base/Create/Update/Response schema hierarchy
   - Field validation rules
   - Serialization config (e.g., `from_attributes`, `class-transformer`)

**Universal principles**:
- Validate all user input at the boundary
- Separate input schemas from response schemas
- Field constraints (min/max length, ranges, patterns)
- Consistent naming with API contract conventions

## Critical Patterns Checklist

- [ ] Rate limiting on expensive endpoints
- [ ] Authentication on protected routes
- [ ] Eager loading for relationships (prevent N+1)
- [ ] Batch operations for bulk writes
- [ ] Transaction blocks for atomic operations
- [ ] Type annotations on all public functions
- [ ] Ownership checks in queries (user can only access their data)
- [ ] No business logic in route handlers (delegate to services)
