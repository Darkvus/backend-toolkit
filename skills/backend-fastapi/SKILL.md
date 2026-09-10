---
name: backend-fastapi
description: "FastAPI coding conventions: router patterns, dependency injection, Pydantic schema standards, repository usage. Use when implementing or reviewing FastAPI code."
---

# FastAPI

Although FastAPI has its own conventions, in this section we will find internal conventions to follow.

## Using explicit field updates on partial updates

Whenever we want to update "x" number of fields of a model instance, we must explicitly specify which fields we want to update instead of persisting the whole object:

```python
resource = await resource_repository.get(id=resource_id)
resource.active = False
resource.deleted_at = datetime.utcnow()
await resource_repository.update(resource, fields=["active", "deleted_at"])
```

Keep in mind that if we have a field that is modified every time an update happens (for example `updated_at`), it will not be updated automatically unless the ORM/DB layer handles it, and must be explicitly specified as well (in the example above it would be: `fields=["active", "deleted_at", "updated_at"]`).

### Reasons

* Avoid "race conditions" from multiple processes that obtain the same instance at the same time and edit different fields. If we don't scope the update to specific fields, the last process that persists will overwrite all fields with its own (possibly stale) values.
* Avoid overworking the DB. By indicating which fields in the DB we want to update, we make the DB work less, since the query to the DB will only be for those fields, and not for all the model's fields.

## Using APIRouter with thin endpoints

Routers must stay thin adapters that delegate to application use cases. Business logic must never live inside the endpoint function.

Correct example:

```python
# FastAPI imports
from typing import Annotated
from fastapi import APIRouter, Depends

# Company imports
from apps.searcher.infrastructure.dependencies import get_search_use_case
from apps.searcher.application.uses_cases.interfaces import ISearchUseCase

router = APIRouter()


@router.post("/travel-options", status_code=200)
def search(
    payload: SearchRequest,
    search_use_case: Annotated[ISearchUseCase, Depends(get_search_use_case)],
):
    return search_use_case.execute(payload)
```

Incorrect example:

```python
@router.post("/travel-options")
def search(payload: SearchRequest):
    # Business logic and data access directly in the endpoint
    results = db.query(Option).filter(Option.origin == payload.origin).all()
    return [r.to_dict() for r in results]
```

## Using FastAPI's dependency injection instead of manual wiring

As much as possible, use `Depends()` to resolve use cases, repositories, and authentication instead of instantiating them by hand inside the endpoint. This keeps endpoints testable and consistent with the layering described in [[backend-chassis-pattern]].

### Repository access (equivalent to `get_queryset`)

Endpoints and use cases must not build queries directly. They must ask the repository for data, and the repository is responsible for applying filters:

```python
def get_resource_repository() -> IResourceRepository:
    return SqlAlchemyResourceRepository()


class ListResourcesUseCase:
    def __init__(
        self,
        resource_repository: Annotated[IResourceRepository, Depends(get_resource_repository)],
    ) -> None:
        self._resource_repository = resource_repository

    def execute(self, filters: dict) -> list[Resource]:
        return self._resource_repository.list(filters=filters)
```

If a use case needs a custom filter applied only in specific scenarios, add it inside the repository method, not by re-implementing queries in the use case:

```python
class SqlAlchemyResourceRepository(IResourceRepository):
    def list(self, filters: dict) -> list[Resource]:
        query = self._base_query()
        if not filters.get("include_archived"):
            query = query.filter(Resource.archived.is_(False))
        return query.filter_by(**filters).all()
```

### Get single object (equivalent to `get_object`)

Whenever we want to get the object referenced by a path parameter, we must go through the repository and raise a domain-level "not found" exception if it doesn't exist, never query the ORM directly in the endpoint:

```python
@router.get("/resources/{resource_id}")
def retrieve(
    resource_id: UUID,
    retrieve_use_case: Annotated[IRetrieveResourceUseCase, Depends(get_retrieve_resource_use_case)],
):
    return retrieve_use_case.execute(resource_id=resource_id)


class RetrieveResourceUseCase:
    def execute(self, resource_id: UUID) -> Resource:
        resource = self._resource_repository.get(id=resource_id)
        if resource is None:
            raise ResourceNotFoundException()
        return resource
```

Incorrect way:

```python
@router.get("/resources/{resource_id}")
def retrieve(resource_id: UUID):
    resource = db.query(Resource).get(resource_id)
```

### Choosing the response/request model per action (equivalent to `get_serializer_class`)

It is very useful to use a different Pydantic model per endpoint/action depending on what is being returned, instead of reusing a single "god" schema for every action.

```python
@router.post("/resources", response_model=ResourceDetailSchema)
def create(payload: ResourceCreateSchema, ...):
    ...


@router.get("/resources", response_model=list[ResourceListItemSchema])
def list_resources(...):
    ...
```

`ResourceCreateSchema` validates and deserializes the input, `ResourceListItemSchema` serializes a lighter representation for listing, and `ResourceDetailSchema` serializes the full representation for creation/retrieval — mirroring how DRF picks a serializer class per action, see [[backend-django-drf]].
