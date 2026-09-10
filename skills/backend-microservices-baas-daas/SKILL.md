---
name: backend-microservices-baas-daas
description: "BaaS and DaaS microservice types: definitions, responsibilities, and when to use each. Use when designing new microservices."
---

# Microservices BAAS and DAAS

The microservices we create must comply with the following construction agreements:

## BAAS (Backend as a Service)

* **Django** models are not used, but **Pydantic** models instead.
* Logic is implemented in Service-type classes, not in Django views. The use of dependency injection is recommended: [python-dependency-injector](https://python-dependency-injector.ets-labs.org/).
* Class-based views that are as "dumb" as possible, meaning they focus on structural validation of incoming data. **GenericViewset** is recommended.
* Use **Company Request** client for all external calls.
* Use logging in key operations and keep in mind that we have middleware that logs input and output of each request.
* Multi-tenant work:
    * We use the Company Travel domain while we find a way to dynamically resolve a tenant's domain for a specific microservice.

## DAAS (Data as a Service)

* Zero business logic, only responsible for CRUD operations.
