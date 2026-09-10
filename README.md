# backend-dev-kit

A [Claude Code](https://claude.com/claude-code) plugin with agents, commands, and skills for Python backend development — Domain-Driven Design architecture, FastAPI, Django REST Framework, and microservice conventions.

## What's included

### Agents

Planning agents that translate Domain-Driven Design architecture into stack-specific implementation plans.

| Agent | Purpose |
|---|---|
| `ddd-planner` | Design or refactor backend architecture using DDD layered architecture (domain, application, infrastructure). |
| `fastapi-planner` | Translate DDD architecture into FastAPI implementation plans (routers, dependency injection, repositories). |
| `djangorestframework-planner` | Translate DDD architecture into Django REST Framework implementation plans with Clean Architecture layering. |
| `daas-planner` | Translate DDD architecture into Django-based Data as a Service (DaaS) implementation plans. |
| `planner-orchestrator` | Consolidate stack-specific plans into actionable engineering tasks and release readiness reports. |

### Commands

Slash commands for common backend workflows.

| Command | Purpose |
|---|---|
| `/explore-plan` | Explore, select a team/agent, plan, and iterate on a user request. |
| `/start-working-on-technical-plan` | Turn a Technical Specification into a consolidated backend implementation plan across microservices. |
| `/create-issues-from-plan` | Create GitHub issues in each affected microservice repository from per-microservice plans. |
| `/create-new-gh-issue` | Create a new GitHub issue for a feature from a context session file. |
| `/start-working-on-issue` | Implement a GitHub issue created by `create-issues-from-plan`. |
| `/update-docstrings` | Replace legacy header docstrings with descriptive module documentation. |
| `/update-permissions` | Synchronize the permissions definition YAML into a microservice. |

### Skills

Convention references that Claude loads automatically when relevant.

| Skill | Purpose |
|---|---|
| `backend-api-design` | REST API conventions: error format, status codes, naming, versioning. |
| `backend-chassis-pattern` | Shared infrastructure and cross-cutting concerns for microservices. |
| `backend-commits` | Git commit message format, types, and scoping rules. |
| `backend-dependency-management` | Version pinning, update strategy, and approval process. |
| `backend-django-drf` | DRF ViewSet, serializer, `@action`, and permission class conventions. |
| `backend-encoding` | Country, language, currency, timezone, and coordinate formats. |
| `backend-fastapi` | FastAPI router, dependency injection, and Pydantic schema conventions. |
| `backend-microservices-baas-daas` | BaaS vs. DaaS microservice types and when to use each. |
| `backend-microservices-directory-structure` | Mandatory DDD directory layout and naming conventions. |
| `backend-permission-management` | RBAC pattern, permission checking, and FastAPI/DRF integration. |
| `backend-units-of-measurement` | Standard units: meters, cents, seconds, UTC datetime. |
| `backend-url-query-params` | Query parameter, pagination, sorting, and filtering conventions. |

## Installation

See [INSTALL.md](INSTALL.md).

## License

No license specified.
