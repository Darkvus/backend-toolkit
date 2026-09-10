---
name: backend-django-drf
description: "Django REST Framework coding conventions: ViewSet patterns, serializer standards, @action usage, permission classes, filter backends. Use when implementing or reviewing DRF code."
---

# Django / DRF

Although Django has its own conventions, in this section we will find internal conventions to follow.

## Using the .save() method of our models

Whenever we want to update "x" number of fields of a model instance, we must explicitly specify to the save() method which fields we want to update:

```python
instance = Foo.objects.get(id=1)
instance.active = False
instance.deleted_at = datetime.utcnow()
instance.save(update_fields=["active", "deleted_at"])
```

Keep in mind that if we have a field that is modified every time .save() is called, for example `models.DateTimeField(auto_now=True)`, it will not be updated and must be explicitly specified as well (in the example above it would be: `instance.save(update_fields=["active", "deleted_at", "updated_at"])`).

### Reasons

* Avoid "race conditions" from multiple processes that obtain the same instance at the same time and edit different fields. If we don't call the `.save()` method with "update_fields", the last process that calls `.save()` will save all its fields to the DB.
* Avoid overworking the DB. By indicating which fields in the DB we want to update, we make the DB work less, since the query to the DB will only be for those fields, and not for all the model's fields.

## Using @action inside GenericViewSets

In GenericViewSets with custom endpoints (outside of CRUD), use the `@action` decorator. This way DRF automatically routes them in the urls (if we use a DRF Router) and we don't need to explicitly indicate them in the urlpatterns, and to know which HTTP verb is used to call that endpoint, we don't need to go to urls.py.

Correct example:

```python
# Django and DRF imports
from rest_framework.routers import SimpleRouter

# Company imports
from apps.searcher.views import SearchViewSet

router = SimpleRouter()
router.register(r"", SearchViewSet, basename="travel-options")

urlpatterns = router.urls
```

In views.py:

```python
@action(methods=("post",), detail=False, url_path="travel-options")
def search(self, request, *args, **kwargs):
  ...
```

Incorrect example:

```python
# Django and DRF imports
from rest_framework.routers import SimpleRouter

# Company imports
from apps.searcher.views import SearchViewSet

router = SimpleRouter()
router.register(r"", SearchViewSet, basename="travel-options")

urlpatterns = [
    path("", include(router.urls)),
    path(
        "search/",
        views.SearchViewSet.as_view({"post": "search"})
    ),
]
```

## GenericViewSets: Using DRF-provided methods

As much as possible, use the methods provided by DRF and don't create our own implementations. For example, if we create a custom endpoint/action, we should keep in mind the methods DRF provides and use them.

### filter_queryset

> "Given a queryset, filter it with whichever filter backend is in use. You are unlikely to want to override this method, although you may need to call it either from a list view, or from a custom `get_object` method if you want to apply the configured filtering backend to the default queryset."

This method is provided by `GenericViewSet`. It filters the queryset with the `filter_backend` currently in use.

If we create a custom endpoint/action and want to get the filtered queryset, we must call this method:

```python
@action(methods=["GET"])
def foo(self, request, *args, **kwargs):
    queryset = self.filter_queryset(self.get_queryset())
    ...
```

Also, if we want to add custom filters that apply, for example, when a certain endpoint (action) is called, or by default depending on the request, we can override this filter:

```python
def filter_queryset(self, queryset):
    if self.action == "foo":
      queryset = queryset.filter()
    if not self.request.user.is_admin:
      queryset = queryset.exclude(only_admin=True)
    return super().filter_queryset(queryset)
```

### get_queryset

> "Get the list of items for this view. This must be an iterable, and may be a queryset. Defaults to using `self.queryset`. This method should always be used rather than accessing `self.queryset` directly, as `self.queryset` gets evaluated only once, and those results are cached for all subsequent requests. You may want to override this if you need to provide different querysets depending on the incoming request. (Eg. return a list of items that is specific to the user)"

This method is provided by `GenericViewSet`. It returns the queryset we are using, by default it returns the value of the `queryset` attribute of our GenericViewSet.

We can override it if we want to provide a different queryset depending on the requested resource.

If we want to get the queryset, we should not access the queryset attribute directly (`self.queryset`) but call this method (`self.get_queryset()`)

```python
@action(methods=["GET"])
def foo(self, request, *args, **kwargs):
    queryset = self.filter_queryset(self.get_queryset())
    ...
```

As we can see, a best practice to get the queryset is to call `self.filter_queryset()` and pass `self.get_queryset()` as a parameter.

### get_object

> "Returns the object the view is displaying. You may want to override this if you need to provide non-standard queryset lookups. Eg if objects are referenced using multiple keyword arguments in the url conf."

This method is provided by `GenericViewSet`. It returns the object we want to get through the URL path.

This method uses `self.filter_queryset(self.get_queryset())` to get the corresponding queryset and filter it, returns a 404 if it doesn't find the object, and checks permissions on that object.

It should be overridden if the object is referenced through multiple parameters in the URL path.

Whenever we want to get the object we reference in the URL, we should call this method:

```python
def retrieve(self, request, *args, **kwargs):
    instance = self.get_object()
```

Incorrect way:

```python
def retrieve(self, request, *args, **kwargs):
    instance = self.get_queryset(id=kwargs.get("id"))
```

### get_serializer_class

> "Return the class to use for the serializer. Defaults to using `self.serializer_class`. You may want to override this if you need to provide different serializations depending on the incoming request. (Eg. admins get full serialization, others get basic serialization)"

This method is provided by `GenericViewSet`. It returns the serializer class (by default `self.serializer_class`).

It is very useful to override it if we use different serializers in the methods/actions/endpoints of our GenericViewSet, since Swagger auto-documents depending on `get_serializer_class`.

In the following example, when a POST / is made, it returns a `FooCreateSerializer` serializer which is what the endpoint will use when calling the `self.get_serializer` method to create the object.

```python
def get_serializer_class(self):
    """
    Return the class to use for the serializer.
    Defaults to using `self.serializer_class`.

    You may want to override this if you need to provide different
    serializations depending on the incoming request.

    (Eg. admins get full serialization, others get basic serialization)
    """

    if self.action == "create":
        return FooCreateSerializer
    return self.serializer_class
```

### get_serializer

> "Return the serializer instance that should be used for validating and deserializing input, and for serializing output."

This method is provided by `GenericViewSet`. It returns the serializer we will use to validate and deserialize the request and to serialize the response.

This method adds context to the serializer obtained through `self.get_serializer_class()`.

When we have a custom method/action/endpoint, we must call this method to get the serializer:

```python
@action(methods=["GET"])
def foo(self, request, *args, **kwargs):
    ...
    serializer = self.get_serializer(instance)
    return Response(serializer.data)
```
