# Update Python File Docstrings

Replace legacy header docstrings with descriptive module documentation.

## Your Task

Scan all Python files in the project and for each file:

1. **Find and remove** the legacy header docstring:
```python
"""
    Project: Demand Responsive Transport - DRT
    Authors: Company Smart Mobility Services S.L.
"""
```

2. **Replace it** with a descriptive 2-line docstring explaining:
   - For regular modules: What the file contains (classes, functions, purpose)
   - For `__init__.py` files: What the package contains and its purpose

## Implementation Steps

1. Use `Glob` to find all `**/*.py` files in the project
2. For each file, use `Read` to check if it contains the legacy docstring
3. If found, analyze the file contents to understand its purpose
4. Use `Edit` to replace the legacy docstring with a new descriptive one

## Docstring Format

Use this format for the replacement:

```python
"""
Brief description of what this module/package contains.
Second line with additional context if needed.
"""
```

## Examples

### Regular module (models.py)
```python
"""
Pydantic models for messaging events and commands.
Defines base message structure, consumers, and configuration.
"""
```

### Schema file (bookings.py)
```python
"""
Event schemas for booking-related messages.
Includes payment, purchase, and transaction events.
"""
```

### Controller (base.py)
```python
"""
Base controller for message ingestion and publishing.
Handles SNS/Kafka channel routing and persistence.
"""
```

### Empty __init__.py
```python
"""
Messaging schemas package.
Contains event and command payload definitions.
"""
```

### __init__.py with imports
```python
"""
Events schema package exports.
Re-exports all event schema classes for external use.
"""
```

## Guidelines

- Keep descriptions concise but informative (2 lines max)
- Focus on WHAT the module contains, not implementation details
- For packages, describe the scope/domain of contained modules
- Use present tense ("Contains...", "Defines...", "Handles...")
- No trailing whitespace in docstrings
- Maintain proper Python formatting

## Execution

Process files systematically:
1. First, list all Python files to process
2. Create a TodoWrite list to track progress
3. Process files in batches, reading and updating each one
4. Report summary when complete (files updated, files skipped)

Start by finding all Python files and identifying which ones have the legacy docstring.