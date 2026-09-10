---
name: backend-dependency-management
description: "Dependency management policy: version pinning, update strategy, and approval process for Python backend services."
---

# Dependency Creation and Management Policy

## Prior Analysis

* Before creating a library, bring it to consensus with the team.
* Ask yourself several questions:
    * Do we have an existing library where the code would fit?
        * Is it necessary to split that library by Poetry "extras"? For example, the same library can be used in BaaS and DaaS but with different dependencies.
    * Is it used in multiple projects?
    * Can it be grouped in a library within a context common to other implementations? (example: AWS SDK, we have implementations of different AWS services).

## Creation

* Register it in our library catalog.
* Remember that we have [Python Library Template](https://github.com/company/company-python-library-template) for creating Python libraries.
* Add new library in [Duck Manager](https://github.com/company/duck-manager/blob/develop/.github/workflows/publish-artifact.yaml) for automatic Artifact publication. Remember that these Artifacts are published in [Nexus](https://nexus.dev.drtcompany.com/#browse/browse:company-pypi-private).

## Management

### Updating New Features

* Review library usage. If we see that the feature adds dependencies that are not necessary in specific contexts, evaluate whether that feature is necessary in that library or the use of Poetry "extras".

### Versioning (x.y.z)

* Increment the **major (x)** when it's a "breaking change", for example when the agreement of the library's external contracts changes.
* Increment the **minor (y)** when we update a dependency that is used in other MSs within the library with "breaking changes". Example: updating Pydantic from version 1.x.x to 2.x.x (Pydantic is used across all projects).
* Increment the **patch (z)** when it's a trivial update, fix (NOT hotfix), adding a new feature, etc.
* Increment the **prerelease (za0)** when it's a hotfix.

## Library Catalog

| **Name** | **Link** |
| --- | --- |
| Django Chassis | [company-django-chassis](https://github.com/company/company-django-chassis) |
| Django Mongoengine | [company-django-mongoengine](https://github.com/company/company-django-mongoengine) |
| FastAPI Chassis | [company-fastapi-chassis](https://github.com/company/company-fastapi-chassis) |
| Requests | [company-requests](https://github.com/company/company-requests) |
| Errors | [company-errors](https://github.com/company/company-errors) |
| Pandora Box SDK | [company-pandora-box-sdk](https://github.com/company/company-pandora-box-sdk) |
| AWS SDK | [company-aws-sdk](https://github.com/company/company-aws-sdk) |
| Celery SDK | [company-celery-sdk](https://github.com/company/company-celery-sdk) |
| Security SDK | [company-security-sdk](https://github.com/company/company-security-sdk) |
| Travels Shift Roadmap DaaS SDK | [company-travels-shift_roadmap-daas-sdk](https://github.com/company/company-travels-shift_roadmap-daas-sdk) |
| i18n SDK | [company-i18n-sdk](https://github.com/company/company-i18n-sdk) |
| Bookings Lead SDK | [company-bookings-lead-sdk](https://github.com/company/company-bookings-lead-sdk) |
| Managers Messaging SDK | [company-managers-messaging-sdk](https://github.com/company/company-managers-messaging-sdk) |
| Realtime Inserter | [company-realtime-inserter](https://github.com/company/company-realtime-inserter) |
