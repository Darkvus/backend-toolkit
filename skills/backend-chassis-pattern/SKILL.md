---
name: backend-chassis-pattern
description: "Chassis pattern for microservices: shared infrastructure, cross-cutting concerns, company-dominus exceptions. Use when setting up new services or reviewing infrastructure patterns."
---

# Chassis Pattern

The chassis pattern provides a foundation for building microservices by implementing cross-cutting concerns once and reusing them across all services. Company maintains two Python chassis implementations:

- **company-django-chassis**: For Django/DRF-based microservices
- **company-fastapi-chassis**: For FastAPI-based microservices

## Cross-Cutting Concerns

Both chassis implementations provide the following capabilities:

| Capability | Description |
|------------|-------------|
| **Distributed Tracing** | OpenTelemetry instrumentation with trace ID propagation |
| **Externalized Configuration** | AWS Secret Manager integration + environment variables |
| **Logging** | JSON structured logging with trace context |
| **Health Check** | `/health-check/` endpoint for monitoring |
| **Version** | `/version/` endpoint exposing service version |
| **API Docs** | Swagger UI (`/`) and ReDoc (`/redoc/`) |
| **Metrics** | HTTP response status counters and request latency |
| **Error Handlers** | Standardized error response format |

### Django-Specific Features

- **Base Model**: UUID primary key, `created_at`, `updated_at`, `active` fields
- **Multi-tenant Support**: via django-tenants
- **Debug Toolbar**: Development debugging support

## Error Response Format

All chassis implementations return errors in a standardized format:

```json
{
    "messages": [
        {
            "type": "ERROR",
            "code": "error_code",
            "description": "Human readable description"
        }
    ],
    "trace_id": "727f672a121bba21a5ad10f7029a6225"
}
```

For validation errors, the format includes field information:

```json
{
    "messages": [
        {
            "type": "ERROR",
            "field": "field_name",
            "details": [
                {
                    "code": "required",
                    "description": "Field required"
                }
            ]
        }
    ],
    "trace_id": "727f672a121bba21a5ad10f7029a6225"
}
```

## Domain Exceptions with company-dominus

The `company-dominus` library provides domain-layer exception classes that integrate with the chassis error handlers. This allows domain logic to raise typed exceptions that are automatically converted to appropriate HTTP responses.

### Exception Types and HTTP Status Codes

| TypeException | HTTP Status Code | Use Case |
|---------------|------------------|----------|
| `VALIDATION` | 400 Bad Request | Input validation failures |
| `BUSINESS` | 400 Bad Request | Business rule violations |
| `PERMISSION` | 403 Forbidden | Authorization failures |
| `TECHNICAL` | 500 Internal Server Error | Infrastructure/system errors |

### Creating Domain Exceptions

Define custom exceptions by extending `DomainBaseException`:

```python
from company_dominus.shared.domain.exceptions import (
    DomainBaseException,
    TypeException,
    LogLevel,
    ErrorDetail,
)


class BookingNotFoundError(DomainBaseException):
    """Raised when a booking cannot be found."""

    code = "booking_not_found"
    description = "The requested booking does not exist"
    type = TypeException.BUSINESS
    log_level = LogLevel.WARNING


class InsufficientCreditsError(DomainBaseException):
    """Raised when user doesn't have enough credits."""

    code = "insufficient_credits"
    description = "User does not have enough credits for this operation"
    type = TypeException.PERMISSION
    log_level = LogLevel.INFO


class InvalidDateRangeError(DomainBaseException):
    """Raised when date range validation fails."""

    code = "invalid_date_range"
    type = TypeException.VALIDATION
    log_level = LogLevel.INFO

    def __init__(self, field: str, start_date: str, end_date: str):
        super().__init__(
            field=field,
            details=[
                ErrorDetail(
                    code="date_range_invalid",
                    description=f"Start date {start_date} must be before end date {end_date}"
                )
            ]
        )
```

### Using Domain Exceptions in Services

```python
from domain.exceptions import BookingNotFoundError, InsufficientCreditsError


class BookingService:
    def __init__(self, booking_repository, user_repository):
        self._booking_repository = booking_repository
        self._user_repository = user_repository

    def cancel_booking(self, booking_id: str, user_id: str) -> None:
        booking = self._booking_repository.find_by_id(booking_id)
        if not booking:
            raise BookingNotFoundError()

        user = self._user_repository.find_by_id(user_id)
        if not user.can_cancel_booking(booking):
            raise InsufficientCreditsError()

        booking.cancel()
        self._booking_repository.save(booking)
```

### Log Levels

Domain exceptions support configurable log levels to control how they appear in logs:

| LogLevel | When to Use |
|----------|-------------|
| `DEBUG` | Development/troubleshooting only |
| `INFO` | Expected business events (validation failures, permission denials) |
| `WARNING` | Unusual but recoverable situations |
| `EXCEPTION` | Unexpected errors requiring investigation (default) |

Set the log level based on whether the exception represents an expected condition or an unexpected error:

```python
class UserNotFoundError(DomainBaseException):
    code = "user_not_found"
    type = TypeException.BUSINESS
    log_level = LogLevel.INFO  # Expected condition, no stack trace needed


class DatabaseConnectionError(DomainBaseException):
    code = "database_connection_failed"
    type = TypeException.TECHNICAL
    log_level = LogLevel.EXCEPTION  # Unexpected, log full stack trace
```

## Installation

### Django Chassis

```toml
[tool.poetry.dependencies]
company-django-chassis = "^1.2.7"
```

### FastAPI Chassis

```toml
[tool.poetry.dependencies]
company-fastapi-chassis = "^0.x.x"
```

Both chassis include `company-dominus` as a transitive dependency.

## Best Practices

1. **Define exceptions in the domain layer**: Keep exception definitions close to the domain logic that raises them.

2. **Use specific exception types**: Create specific exceptions rather than generic ones to improve error handling and debugging.

3. **Choose appropriate TypeException**:
   - `VALIDATION` for input/format errors
   - `BUSINESS` for domain rule violations
   - `PERMISSION` for authorization failures
   - `TECHNICAL` for infrastructure errors

4. **Set meaningful log levels**: Use `INFO` for expected conditions, `EXCEPTION` for unexpected errors.

5. **Never expose internal details**: Exception descriptions should be user-facing. Don't include stack traces, SQL queries, or internal identifiers in the `description` field.

6. **Use error codes for client handling**: Clients should rely on `code` values for programmatic error handling, not on description text which may change.
