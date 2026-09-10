---
name: backend-api-design
description: "REST API design conventions: error response format, HTTP status codes, endpoint naming, versioning. Use when designing or reviewing API endpoints."
---

# API Design

## Error Responses

### Best Practices

1. Always use the same response structure.
2. Use **JSON** format.
3. Define error severity levels. Example: INFO, WARNING, ERROR and FATAL.
4. Use container properties for errors and make it a list.
5. Include **alias** or **code** specific to each error.
6. Include descriptive messages for each error so that the API is understandable by humans.
7. In external APIs, an error code will normally be returned to avoid providing too much information about what happened.

### Our Convention

#### Status 400 - Bad Request

```json
{
    "messages": [
        {
            "type": "INFO",
            "field": "name",
            "details": [
                {
                    "code": "required",
                    "description": "This Field Is Required."
                }
            ]
        },
        {
            "type": "INFO",
            "field": "geometry_area",
            "details": [
                {
                    "code": "required",
                    "description": "This Field Is Required."
                }
            ]
        },
        {
            "type": "INFO",
            "field": "zones",
            "details": [
                {
                    "code": "does_not_exist",
                    "description": "Invalid Pk \"8916Ed92-Bdda-462B-B125-Ddd39Ad24680\" - Object Does Not Exist."
                }
            ]
        }
    ],
    "trace_id": 189523488292774445242427953427527174528
}
```

#### Status 400 - Bad Request with Custom Response Code

Structure used primarily in **external** APIs.

```json
{
    "messages": [
        {
            "type": "ERROR",
            "code": "600",
            "description": "Not found."
        }
    ],
    "trace_id": 189523488292774445242427953427527174528
}
```

The response code (`code`) must be mapped in the company-errors library, and if possible in the functional document of the respective API or MS.

**Considerations:**

* The **field** property is included to indicate the field where the error occurs.
* Error codes are those defined by the frameworks used: Django and Django Rest Framework.

| **Code** | **Description** |
| --- | --- |
| required | This Field Is Required |
| does_not_exist | Invalid Pk "8916Ed92-Bdda-462B-B125-Ddd39Ad24680" - Object Does Not Exist. |
| max_length | Ensure This Field Has No More Than 50 Characters. |
| invalid | A Valid Integer Is Required. |

#### Status 500 - Server Internal Error

```json
{
    "messages": [
        {
            "type": "ERROR",
            "code": "CEL001",
            "description": "Internal Error Simulation"
        }
    ]
}
```

---

## Throwing Exceptions from Microservices

To throw errors that have the error format used in the microservices chassis, we throw framework specific exceptions passing the information we need.

For example, to throw an error in a validation it can be implemented as follows:

We define the type of exception we want to throw. The HTTP code returned will depend on the type of exception we throw, in this case a ParseError exception is used which returns HTTP 400. For example, APIException returns HTTP 500.

More information at [DRF Exceptions](https://www.django-rest-framework.org/api-guide/exceptions/)

```python
from rest_framework.exceptions import ParseError


class UserValidationError(ParseError):
    """UserValidationError error"""
    pass
```

To throw the exception:

```python
if res.status_code == 404 or not content["active"]:
    raise UserValidationError(code="100", detail="Passenger not found.")
```

---

## Error Codes

Error codes are centralized in the [company-errors](https://github.com/company/company-errors) library.

These codes are used by the different MSs to return the same code in a common way.

```python
class UserErrorCodes(BaseErrorCodeEnum):
    """
    Enum class to group all error codes for User context
    """

    IDP_NOT_FOUND = ("0100", "User with this identity_provider not found")
    NOT_ALLOWED = ("0101", "User not allowed")
    DISABLED = ("0110", "User is disabled")
    BANNED = ("0111", "User is banned")
    GROUP = ("0112", "Does not belongs to the group")
    ALREADY_EXISTS = ("0113", "User already exists")
    CREATION_ERROR = ("0114", "An error occurred during the User creation process")
    DELETE_RELATED_ERROR = ("0115", "An error occurred during the User deletion process")
    LEGAL_CHECK_NOT_FOUND = ("0116", "Legal check not found in update profile")
    LEGAL_CHECK_MANDATORY = ("0117", "Legal check is mandatory you must accept it")
    EMAIL_REQUIRED = ("0118", "Email is required")
    STAGE_NOT_IN_REGISTER = ("0119", "The stage of the user is not IN_REGISTER")
    NOT_CANCEL_ACCOUNT_BY_TRIP = ("0120", "You have future trips you cannot cancel the account")
    STAGE_IS_IN_REGISTER = ("0121", "The stage of the user is in IN_REGISTER")
    COUNTRY_CODE_REQUIRED = ("0122", "Country code is required")
    PHONE_REQUIRED = ("0123", "Phone is required")
    GENERIC_ERROR = ("0199", "An User generic error")


# ... (remaining error code classes are already in English)
```

For the complete list of error codes, see the [company-errors](https://github.com/company/company-errors) library.

### Logger Errors

```json
{
  "0x404": "The Shift is not there, and the DB hasn't seen it either. Make sure the Shift is back from vacation before requesting it again."
}

{
  "code": "0x4004",
  "description": "strategy not implement exception"
}
```
