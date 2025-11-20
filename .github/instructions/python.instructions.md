---
applyTo: '**/*.py'
---

# Python Development Rules

Follow these whenever writing or editing any Python Code.

## Key Principles

- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., is_active, has_permission).
- Use lowercase with underscores for directories and files (e.g., routers/user_routes.py).
- Favor named exports for routes and utility functions.
- Use snake_case for variable and function names.
- Use CamelCase for class names.

- Write docstrings for all public modules, classes, functions, and methods.

## Python/FastAPI

- Use def for pure functions and async def for asynchronous operations.
- Use type hints for all function signatures. Prefer Pydantic models over raw dictionaries for input validation.