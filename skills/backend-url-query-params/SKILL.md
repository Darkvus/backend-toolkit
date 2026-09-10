---
name: backend-url-query-params
description: "URL query parameter and filter conventions: naming, pagination, sorting, and filtering patterns for REST APIs. Use when implementing query parameters or filters."
---

# URL Query Params Filters

Django uses the following expression to filter with the ORM: `field_name__lookup_expression`, which is very Django ORM-specific.

In our interface agreements, we have more universal filter names. For example, to filter by a date we use `from_datetime`, which translates in Django to `datetime__gte`. To filter by different statuses (status) we use `statuses`, which in Django would be `status__in`.

We want to change all BaaS and DaaS to use the same convention, meaning they should not use Django's `__lookup_expression`.

| **Filter** | **Convention** | **Lookup** |
| --- | --- | --- |
| From (inclusive) >= | from\_ | \_\_gte |
| From (strict) > | above\_ | \_\_gt |
| To (inclusive) <= | to\_ | \_\_lte |
| To (strict) < | below\_ | \_\_lt |
| Array options | fieldS/ES (Plural) | \_\_in |
| Relation/identifier (FK, O2O) | Don't send \_id |  |
| Check if a field is null | \_empty → for lists [] \_isnull → for single field | \_\_isnull |

To do this, we will create filter classes inheriting from our class `from company_django.filtersets import BaseFilterSet`

Example:

```python
# Django and DRF imports
from django_filters import filters

# Company imports
from company_django.filtersets import BaseFilterSet
from apps.configuration.models.vehicle import Vehicle


class VehicleFilterSet(BaseFilterSet):
    """
    FilterSet for Vehicle
    """

    plate = filters.CharFilter(field_name="plate", lookup_expr="icontains")
    partners = filters.CharFilter(field_name="partner", lookup_expr="in")
    ids = filters.BaseInFilter(field_name="id")

    class Meta:
        model = Vehicle
        fields = ["partner", "plate", "partners", "ids"]


class PartnerFilterSet(BaseFilterSet):
    """
    FilterSet for Partner
    """

    name = filters.CharFilter(field_name="name", lookup_expr="icontains")
    ids = filters.BaseInFilter(field_name="id")
    vehicles = filters.CharFilter(field_name="partner_vehicles", lookup_expr="in")

    class Meta:
        model = Vehicle
        fields = ["name", "ids", "vehicles"]
```

The same convention applies to FastAPI services. Query params are declared explicitly on the endpoint (or grouped in a Pydantic filter schema) and translated to the repository/ORM lookup inside the repository, never inside the endpoint:

```python
# FastAPI imports
from typing import Annotated
from fastapi import APIRouter, Depends, Query
from pydantic import BaseModel

router = APIRouter()


class VehicleFilters(BaseModel):
    plate: str | None = None
    partners: list[str] | None = None
    ids: list[str] | None = None
    from_created_at: datetime | None = None
    to_created_at: datetime | None = None


@router.get("/vehicles")
def list_vehicles(
    filters: Annotated[VehicleFilters, Query()],
    list_use_case: Annotated[IListVehiclesUseCase, Depends(get_list_vehicles_use_case)],
):
    return list_use_case.execute(filters=filters.model_dump(exclude_none=True))


class SqlAlchemyVehicleRepository(IVehicleRepository):
    def list(self, filters: dict) -> list[Vehicle]:
        query = self._base_query()
        if plate := filters.get("plate"):
            query = query.filter(Vehicle.plate.ilike(f"%{plate}%"))
        if partners := filters.get("partners"):
            query = query.filter(Vehicle.partner.in_(partners))
        if ids := filters.get("ids"):
            query = query.filter(Vehicle.id.in_(ids))
        if from_created_at := filters.get("from_created_at"):
            query = query.filter(Vehicle.created_at >= from_created_at)
        if to_created_at := filters.get("to_created_at"):
            query = query.filter(Vehicle.created_at <= to_created_at)
        return query.all()
```
