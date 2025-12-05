# ADR 0003: DATABASE_URL Format Requirement

**Date**: 2025-12-05  
**Status**: Accepted  
**Context**: Django startup configuration

## Decision

Use `DATABASE_URL` environment variable with `dj-database-url` parser instead of individual database configuration variables.

## Context

Django 5.2.8 with Python 3.13.7 experienced indefinite hangs during `django.setup()` when using individual database configuration variables (DB_NAME, DB_USER, DB_PASSWORD, DB_HOST, DB_PORT). The hang occurred before any application code executed, preventing all Django management commands from running.

## Investigation

- Direct psycopg2 connections worked correctly
- Django with SQLite configuration worked correctly
- Issue was specific to PostgreSQL configuration parsing during Django initialization
- Environment variables loaded correctly via python-decouple
- No circular imports or code issues identified

## Solution

Implement `DATABASE_URL` format using `dj-database-url` library:

```python
DATABASE_URL = config("DATABASE_URL", default=None)

if DATABASE_URL:
    DATABASES = {
        "default": dj_database_url.parse(
            DATABASE_URL,
            conn_max_age=600,
            conn_health_checks=True,
        )
    }
```

**Format**: `postgresql://USER:PASSWORD@HOST:PORT/DATABASE`

## Consequences

### Positive
- Django startup works correctly
- Simpler configuration (single variable vs 5 variables)
- Standard format used by many platforms (Heroku, Vercel, etc.)
- Maintains all connection pooling and health check features

### Negative
- Requires additional dependency: `dj-database-url`
- Passwords with special characters must be URL-encoded

### Neutral
- Fallback to individual variables maintained for compatibility
- Supabase pooler port (6543) required instead of direct port (5432)

## Alternatives Considered

1. **Downgrade Python/Django**: Would lose latest features and security updates
2. **Use SQLite**: Not suitable for production service marketplace
3. **Docker isolation**: Adds complexity without solving root cause
4. **Individual variables only**: Caused indefinite Django startup hang

## References

- [See: ../../deployment/common-configuration.md#database-configuration]
- Library: https://github.com/jazzband/dj-database-url
