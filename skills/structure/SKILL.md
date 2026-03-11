---
name: structure
description: Enforce proper project structure — separation of concerns, modularity, and reusability. Use when user says /structure, wants to audit project structure, needs guidance on where to put new code, or wants to restructure a disorganized codebase. Respects opinionated framework conventions.
---

# Project Structure Enforcement

Audit and enforce separation of concerns, modularity, and reusability in project structure.

## Usage

```
/structure              # Audit current project structure
/structure <path>       # Audit a specific directory
/structure where <desc> # Where should this new code go?
```

## Core Principle

**If the framework is opinionated, follow its conventions.** Only apply custom layering to unopinionated frameworks or projects without a framework.

## Step 1: Detect the Framework

Before any analysis, identify what you're working with:

### Opinionated Frameworks (follow their structure)

| Framework | Convention | Do NOT fight it |
|-----------|-----------|-----------------|
| **Rails** | `app/models/`, `app/controllers/`, `app/services/`, `app/views/` | MVC is the law |
| **Django** | `<app>/models.py`, `<app>/views.py`, `<app>/urls.py`, `<app>/serializers.py` | App-based modular structure |
| **Next.js** | `app/` (App Router) or `pages/`, `components/`, `lib/` | File-based routing |
| **Angular** | Feature modules, `*.component.ts`, `*.service.ts`, `*.module.ts` | Module-based architecture |
| **NestJS** | `*.module.ts`, `*.controller.ts`, `*.service.ts`, `*.dto.ts` | Decorator-based DI modules |
| **Laravel** | `app/Http/Controllers/`, `app/Models/`, `app/Services/` | MVC + service layer |
| **Phoenix** | `lib/<app>/`, `lib/<app>_web/`, contexts | Context-based boundaries |
| **Spring Boot** | `controller/`, `service/`, `repository/`, `model/`, `dto/` | Layered + DI |
| **SvelteKit** | `src/routes/`, `src/lib/` | File-based routing |
| **Nuxt** | `pages/`, `components/`, `composables/`, `server/` | Directory conventions |

For these: read the framework docs/conventions and enforce THOSE, not a generic pattern.

### Unopinionated Frameworks (apply layered architecture)

| Framework | Needs Structure Guidance |
|-----------|------------------------|
| Express / Koa / Hono | Yes — no prescribed structure |
| FastAPI / Flask / Starlette | Yes — flat by default |
| Go stdlib / Gin / Chi / Echo | Yes — Go has conventions but not enforced |
| Rust Actix / Axum | Yes — no prescribed structure |
| Sinatra | Yes — minimal structure |

For these: apply the layered architecture below.

## Step 2: Layered Architecture (for unopinionated stacks)

### The Layers

```
Routes/Handlers    → HTTP concerns only (parse request, call service, return response)
     ↓
Services           → Business logic (orchestration, rules, validation)
     ↓
Data Access        → Database queries, external API calls, file I/O
     ↓
Models/Entities    → Data structures, types, schemas
```

### Rules

1. **Routes/Handlers** should be thin — parse input, call a service, format output. No business logic, no direct DB queries.
2. **Services** contain business logic. They don't know about HTTP (no request/response objects). They receive typed parameters and return typed results.
3. **Data access** is isolated. Services call repository/data functions, never construct queries directly.
4. **Models** are plain data structures. No business logic in models (unless using a rich domain model pattern intentionally).
5. **Each layer only calls the layer below it.** Never skip layers (handler calling DB directly) or call upward (service importing a route).

### Cross-Cutting Concerns

These live outside the main layers:

| Concern | Where It Goes |
|---------|--------------|
| Authentication middleware | Middleware/interceptor layer, before routes |
| Authorization | Service layer (business rule) or middleware (role check) |
| Logging | Middleware for request logging, service layer for business events |
| Validation | Input validation at route boundary, business validation in services |
| Error handling | Centralized error handler middleware, domain errors in services |
| Configuration | Dedicated config module, loaded once at startup |
| Shared types/interfaces | Shared types module, imported by any layer |

## Step 3: Audit

When auditing, check for these violations:

### Separation of Concerns Violations

| Violation | Where to Find It | Fix |
|-----------|-----------------|-----|
| Business logic in route handler | Grep for DB queries, complex conditionals, or business rules in route/controller files | Extract to service |
| HTTP concerns in service | Grep for `req`, `res`, `request`, `response`, HTTP status codes in service files | Service should return data, let handler format response |
| DB queries scattered everywhere | Grep for query builder / ORM calls outside data access layer | Centralize in repository/data access module |
| Config values hardcoded | Grep for magic strings, URLs, credentials in business logic | Extract to config module |

### Modularity Violations

| Violation | How to Detect | Fix |
|-----------|--------------|-----|
| God module / god file | File > 500 lines or exports > 15 functions | Split by responsibility |
| Circular dependencies | Import cycles (A imports B, B imports A) | Extract shared code to a third module |
| Tight coupling | Module directly instantiates its dependencies | Use dependency injection or factory pattern |
| Feature code spread across unrelated directories | Related feature code in 5+ different top-level folders | Co-locate by feature (vertical slicing) |

### Reusability Violations

| Violation | How to Detect | Fix |
|-----------|--------------|-----|
| Duplicated logic | Grep for similar code patterns across files | Extract to shared utility or service |
| Utilities mixed with business logic | General-purpose functions in domain-specific files | Move to utils/helpers module |
| No shared types | Same data shape defined differently in multiple places | Create shared type/interface definitions |

## Audit Output Format

```
## Structure Audit

**Framework detected:** [name] ([opinionated/unopinionated])
**Convention followed:** [framework convention / layered architecture]

### Violations Found

**Separation of Concerns (X issues)**
- [file:line] — Business logic in route handler: [description]
- [file:line] — DB query in controller: [description]

**Modularity (X issues)**
- [file:line] — God module (N lines, M exports): suggest splitting by [responsibility]
- [file:line] — Circular dependency: A → B → A

**Reusability (X issues)**
- [file1:line] and [file2:line] — Duplicated logic: [description]

### Recommendations
1. [Highest priority fix]
2. [Second priority fix]
3. [Third priority fix]
```

## "Where Should This Go?" Decision Tree

When the user asks where to put new code:

```
Is this HTTP/request handling? → Route/Controller/Handler
Is this business logic?       → Service
Is this a DB query?           → Data access / Repository
Is this a data structure?     → Model / Entity / Schema
Is this reusable utility?     → Utils / Helpers / Lib
Is this middleware?            → Middleware directory
Is this configuration?        → Config module
Is this a type/interface?     → Types / Shared module
```

If the project uses an opinionated framework, adapt the answer to match its conventions. For example, in Django the answer to "where does business logic go?" might be "in the view or a dedicated service module in the app" depending on the project's established patterns.

## What This Skill Does NOT Do

- **Does not scaffold new projects** — that's a separate concern
- **Does not prescribe folder names** — it enforces principles, names follow the framework or existing conventions
- **Does not force OOP or functional** — separation of concerns applies to both paradigms
- **Does not restructure without the user's approval** — audit and recommend, don't auto-refactor
