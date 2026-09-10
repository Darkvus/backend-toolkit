---
name: backend-microservices-directory-structure
description: "Mandatory directory structure for backend microservices: DDD folder layout, actor separation, naming conventions, and module organization. Use when creating new services or reviewing project structure."
---

# Microservices Internal Directory Structure

Within the different MSs we follow a DDD (Domain-Driven Design) layered architecture organized by actors and modules.

## Target DDD Structure

```
apps/
├── {actor}/                          # internal, passenger, driver, operator, public
│   ├── {module}/                     # trip, shift, booking, etc. (based on aggregate)
│   │   ├── api/
│   │   │   └── v1/
│   │   │       ├── serializers.py
│   │   │       ├── urls.py
│   │   │       └── views.py
│   │   ├── application/
│   │   │   ├── services/
│   │   │   │   ├── dtos.py
│   │   │   │   └── {service_name}_service.py
│   │   │   └── use_cases/
│   │   │       ├── dtos.py
│   │   │       ├── {feature_A}_use_case.py
│   │   │       └── {feature_B}_use_case.py
│   │   ├── domain/
│   │   │   ├── aggregates.py
│   │   │   ├── entities.py
│   │   │   ├── value_objects.py
│   │   │   └── repositories.py        # Interfaces only
│   │   └── infrastructure/
│   │       ├── services/
│   │       └── repositories/          # Implementations
│   └── shared/                        # Cross-module concerns within actor
│       ├── application/
│       ├── domain/
│       └── infrastructure/
├── internal/
│   ├── events/                        # Event consumers
│   └── commands/                      # Command handlers
└── shared/                            # Cross-actor concerns
    ├── application/
    ├── domain/
    └── infrastructure/
```

## Layer Responsibilities

| Layer | Purpose | Dependencies |
|-------|---------|--------------|
| **api/** | HTTP adapters (serializers, views, URLs) | application |
| **application/** | Use cases, DTOs, orchestration | domain |
| **domain/** | Aggregates, entities, value objects, repository interfaces | None (pure Python) |
| **infrastructure/** | Repository implementations, external services, models | domain, application |

**Dependency Rule:** Dependencies flow inward only: `api → application → domain ← infrastructure`

## Modules Definition

A module is defined by its main aggregate. For different operations on an aggregate, create specialized aggregate classes:

```python
from company_dominus.shared.domain.aggregate import AggregateModel

class ShiftDetailAggregate(AggregateModel):
    """For read detail operations"""
    ...

class ShiftListAggregate(AggregateModel):
    """For list operations"""
    ...

class ShiftCreateAggregate(AggregateModel):
    """For create operations"""
    ...

class ShiftUpdateAggregate(AggregateModel):
    """For update operations"""
    ...
```

## Repository Pattern

Repository interfaces live in `domain/`, implementations in `infrastructure/repositories/`:

```python
# domain/repositories.py
import abc
from typing import Optional
from uuid import UUID

from company_dominus.shared.domain.criteria import Criteria

class IRideHailingTripRepository(abc.ABC):
    @abc.abstractmethod
    def retrieve(self, _id: UUID) -> Optional[RideHailingTripAggregate]:
        ...

    @abc.abstractmethod
    def update(self, ride_hailing_trip: RideHailingTripAggregate) -> None:
        ...

    @abc.abstractmethod
    def matching(self, criteria: Optional[Criteria] = None) -> MatchingResults[RideHailingTripAggregate]:
        ...

    @abc.abstractmethod
    def delete(self, _id: UUID) -> None:
        ...
```

The `Criteria` class from `company-dominus` provides:
- `Filtering` - Field-based filtering with name/value pairs
- `Ordering` - Sort order with ASC/DESC support
- `Pagination` - Limit/offset pagination

### Custom Criteria

Each microservice should implement its own specific criteria classes inheriting from the base `Criteria`. Place them in `domain/criteria.py`:

```python
# domain/criteria.py
from uuid import UUID

from company_dominus.shared.domain.criteria import (
    Criteria,
    Filtering,
    FilteringField,
)


class PartnerWithPartnerExternalCriteria(Criteria):
    """
    Partner with Partner External Criteria
    """

    @staticmethod
    def create(partner_id: UUID, partner_external_id: str):
        """
        Create Partner with Partner External Criteria
        """

        filters = {
            "partner": str(partner_id),
            "partner_external_id": partner_external_id,
        }

        filtering = Filtering(
            fields=[
                FilteringField(name=key, value=str(value))
                for key, value in filters.items()
            ]
        )

        return PartnerWithPartnerExternalCriteria(filters=filtering)
```

This pattern provides:
- **Type-safe criteria construction** - Factory methods with explicit parameters
- **Domain-specific naming** - Criteria classes named after their business purpose
- **Encapsulated filter logic** - Filter construction details hidden from callers

## Actors

We divide the codebase by actor. Typical actors in Company are:
- `internal` - Internal/system operations
- `operation` - Operator/backoffice users
- `passenger` - Passenger-facing APIs
- `driver` - Driver-facing APIs
- `public` - Public/unauthenticated endpoints

The "operation", "passenger" and "driver" actors use corresponding authentication classes from company-security-sdk (`OperatorAuthentication`, `PassengerAuthentication`, `DriverAuthentication`).

**Important:** Nothing corresponding to another actor can be imported between actors. For example, from passenger views you can only import from the passenger actor's modules.

## URL Prefixes by Actor

URLs require the prefix according to the actor/package (this prefix will be set in the different `router.register`):

| Actor | URL Prefix |
| --- | --- |
| Public | /public/... |
| Internal | /internal/... |
| Operator | /operation/... |
| Driver | /driver/... |
| Passenger | /... |

## Path Format

This way, endpoints will have the following path:

```
/{context}/{ms-type}/{ms-name}/api/v1/{actor/package}/{viewset-path}/
```

Example:

```
/users/baas/passenger/api/v1/operation/passengers/
```
