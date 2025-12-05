# Common Deployment Configuration

This document describes the common configuration patterns used across deployment environments in the HomeSer service marketplace, ensuring consistency and maintainability.

## Overview

The HomeSer deployment configuration follows consistent patterns for environment management, service connectivity, and platform-specific configurations. This document outlines the standard configuration approaches for different deployment environments.

## Environment Management Patterns

### Environment Variable Structure
All configurations follow the naming convention: `{CATEGORY}_{SETTING_NAME}`

**Categories:**
- `DB_` - Database related configurations
- `REDIS_` - Redis/cache related configurations
- `SUPABASE_` - Supabase service configurations  
- `CLOUDINARY_` - Cloudinary service configurations
- `SSL_` - SSLCommerz payment configurations
- `EMAIL_` - Email service configurations
- `DJANGO_` - Django framework configurations
- `FRONTEND_` - Frontend specific configurations

### Environment Validation
- All required environment variables are validated at startup
- Default values provided for development when appropriate
- Configuration validation prevents runtime failures
- Error messages provide clear guidance for missing variables

## Database Configuration

### Supabase PostgreSQL Settings

**Primary Method: DATABASE_URL**
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
    DATABASES["default"]["OPTIONS"] = {
        "connect_timeout": 10,
        "options": "-c statement_timeout=30000",
    }
```

**Format:** `postgresql://USER:PASSWORD@HOST:PORT/DATABASE`
**Example:** `postgresql://postgres.xxx:pass@aws-1-ap-southeast-1.pooler.supabase.com:6543/postgres`

**Important:** Use Supabase connection pooler port (6543), not direct port (5432)

**Fallback Method: Individual Settings**
```python
else:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': config('DB_NAME'),
            'USER': config('DB_USER'),
            'PASSWORD': config('DB_PASSWORD'),
            'HOST': config('DB_HOST'),
            'PORT': config('DB_PORT', default='6543'),
            'CONN_MAX_AGE': 600,
            'CONN_HEALTH_CHECKS': True,
            'OPTIONS': {
                'connect_timeout': 10,
                'options': '-c statement_timeout=30000',
            }
        }
    }
```

### Connection Pooling
- **CONN_MAX_AGE**: 600 seconds - connection reuse duration
- **CONN_HEALTH_CHECKS**: True - validates connection before reuse
- **connect_timeout**: 10 seconds - connection establishment timeout
- **statement_timeout**: 30000ms - query execution timeout
        'TEST': {
            'NAME': config('DB_TEST_NAME', default='test_homeser'),
        }
    }
}
```

**Configuration Parameters:**
- `DB_CONN_MAX_AGE`: Connection persistence (default 600 seconds)
- `DB_MIN_CONN`, `DB_MAX_CONN`: Connection pool sizes
- `DB_SSL_MODE`: SSL enforcement for database connections
- `DB_TIMEOUT`: Connection timeout settings

### Database Migration Strategy
- Automated migrations during deployment
- Backup procedures before migrations
- Rollback capabilities for failed migrations
- Testing of migrations in staging environment first

## Caching Configuration

### Redis Settings
```python
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': config('REDIS_URL'),
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
            'CONNECTION_POOL_KWARGS': {
                'max_connections': config('REDIS_MAX_CONN', cast=int, default=20),
                'retry_on_timeout': True,
            }
        },
        'KEY_PREFIX': 'homeser',
        'TIMEOUT': 900,  # 15 minutes hardcoded
    }
}
```

**Cache Configuration Parameters:**
- `REDIS_URL`: Connection string for Redis instance
- `REDIS_MAX_CONN`: Maximum connection pool size (default 20)
- `CACHE_KEY_PREFIX`: Namespace for cache keys (default 'homeser')
- `CACHE_TIMEOUT`: Hardcoded to 900 seconds (15 minutes) - not configurable via environment

### Pagination Configuration
The application uses predefined pagination limits defined in `homeser/constants.py`:

```python
DEFAULT_PAGE_SIZE = 20      # Default items per page
MAX_PAGE_SIZE = 100         # Maximum allowed page size (for security)
REVIEWS_PER_PAGE = 10       # Reviews per page in service detail views
BOOKINGS_PER_PAGE = 20      # Bookings per page in booking lists
PAYMENTS_PER_PAGE = 20      # Payments per page in payment history
```

**Pagination Parameters:**
- `DEFAULT_PAGE_SIZE`: Default number of items per page (20)
- `MAX_PAGE_SIZE`: Maximum allowed page size to prevent resource exhaustion (100)
- `REVIEWS_PER_PAGE`: Number of reviews shown per page (10)
- `BOOKINGS_PER_PAGE`: Number of bookings shown per page (20)
- `PAYMENTS_PER_PAGE`: Number of payments shown per page (20)

### Cache Strategies
- **Short-term cache**: 5-15 minutes for frequently changing data
- **Medium-term cache**: 30-60 minutes for semi-static data
- **Long-term cache**: 2-24 hours for static reference data
- **Session cache**: Appropriate timeouts for user sessions

## API and Service Configuration

### CORS Settings
```python
CORS_ALLOWED_ORIGINS = config('CORS_ALLOWED_ORIGINS', cast=lambda v: [s.strip() for s in v.split(',')])
CORS_ALLOW_CREDENTIALS = True
CORS_ALLOW_ALL_ORIGINS = config('CORS_ALLOW_ALL_ORIGINS', cast=bool, default=False)
```

**Configuration Parameters:**
- `CORS_ALLOWED_ORIGINS`: List of allowed frontend origins
- `CORS_ALLOW_CREDENTIALS`: Enable credential sharing
- `CORS_ALLOW_ALL_ORIGINS`: Development override setting

### Rate Limiting
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': config('ANON_RATE_LIMIT', default='100/hour'),
        'user': config('USER_RATE_LIMIT', default='1000/hour'),
    }
}
```

**Rate Limiting Parameters:**
- `ANON_RATE_LIMIT`: Anonymous user rate limit
- `USER_RATE_LIMIT`: Authenticated user rate limit
- Custom limits for sensitive endpoints

## Authentication Configuration

### JWT Settings
```python
from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=config('JWT_ACCESS_TOKEN_MINUTES', cast=int, default=15)),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=config('JWT_REFRESH_TOKEN_DAYS', cast=int, default=7)),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'UPDATE_LAST_LOGIN': False,
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': config('DJANGO_SECRET_KEY'),
    'VERIFYING_KEY': None,
    'AUDIENCE': None,
    'ISSUER': None,
    'JWK_URL': None,
    'LEEWAY': 0,
    'AUTH_HEADER_TYPES': ('Bearer',),
    'AUTH_HEADER_NAME': 'HTTP_AUTHORIZATION',
    'USER_ID_FIELD': 'id',
    'USER_ID_CLAIM': 'user_id',
    'USER_AUTHENTICATION_RULE': 'rest_framework_simplejwt.authentication.default_user_authentication_rule',
    'AUTH_TOKEN_CLASSES': ('rest_framework_simplejwt.tokens.AccessToken',),
    'TOKEN_TYPE_CLAIM': 'token_type',
    'TOKEN_USER_CLASS': 'rest_framework_simplejwt.models.TokenUser',
    'JTI_CLAIM': 'jti',
    'SLIDING_TOKEN_REFRESH_EXP_CLAIM': 'refresh_exp',
    'SLIDING_TOKEN_LIFETIME': timedelta(minutes=5),
    'SLIDING_TOKEN_REFRESH_LIFETIME': timedelta(days=1),
}
```

**JWT Configuration Parameters:**
- `JWT_ACCESS_TOKEN_MINUTES`: Access token lifetime (default 15 minutes)
- `JWT_REFRESH_TOKEN_DAYS`: Refresh token lifetime (default 7 days)
- `JWT_SIGNING_KEY`: JWT signing key (from Django SECRET_KEY)

## Third-Party Service Configuration

### Supabase Configuration
```python
SUPABASE_URL = config('SUPABASE_URL')
SUPABASE_KEY = config('SUPABASE_KEY')
SUPABASE_JWT_SECRET = config('SUPABASE_JWT_SECRET')
SUPABASE_AUTO_LOGIN = config('SUPABASE_AUTO_LOGIN', cast=bool, default=False)
```

### Email Configuration
```python
EMAIL_BACKEND = config('EMAIL_BACKEND', default='django.core.mail.backends.smtp.EmailBackend')
EMAIL_HOST = config('EMAIL_HOST')
EMAIL_PORT = config('EMAIL_PORT', default=587, cast=int)
EMAIL_HOST_USER = config('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = config('EMAIL_HOST_PASSWORD')
EMAIL_USE_TLS = config('EMAIL_USE_TLS', default=True, cast=bool)
DEFAULT_FROM_EMAIL = config('DEFAULT_FROM_EMAIL')
```

**Email Configuration Parameters:**
- `EMAIL_HOST`: SMTP server host
- `EMAIL_PORT`: SMTP server port (default 587)
- `EMAIL_HOST_USER`: SMTP authentication username
- `EMAIL_HOST_PASSWORD`: SMTP authentication password
- `EMAIL_USE_TLS`: Enable TLS encryption (default True)
- `DEFAULT_FROM_EMAIL`: Default sender email address
- `EMAIL_BACKEND`: Django email backend (default SMTP)

## Business Configuration

### Business Rule Settings
The application includes configurable business rules and rates:

```python
TAX_RATE = config('TAX_RATE', cast=float, default=0.15)  # 15% default tax
EXPRESS_FEE = config('EXPRESS_FEE', cast=float, default=0.05)  # 5% express fee
PRIORITY_SUPPORT_COST = config('PRIORITY_SUPPORT_COST', cast=float)
WARRANTY_COST_PER_MONTH = config('WARRANTY_COST_PER_MONTH', cast=float)
BUSINESS_SMALL_DISCOUNT = config('BUSINESS_SMALL_DISCOUNT', cast=float)
BUSINESS_MEDIUM_DISCOUNT = config('BUSINESS_MEDIUM_DISCOUNT', cast=float)
BUSINESS_LARGE_DISCOUNT = config('BUSINESS_LARGE_DISCOUNT', cast=float)
GOVERNMENT_DISCOUNT = config('GOVERNMENT_DISCOUNT', cast=float)
```

**Business Parameters:**
- `TAX_RATE`: Tax rate applied to bookings (default 15%)
- `EXPRESS_FEE`: Additional fee for express services (default 5%)
- `PRIORITY_SUPPORT_COST`: Additional cost for priority support
- `WARRANTY_COST_PER_MONTH`: Warranty cost per month
- `BUSINESS_*_DISCOUNT`: Tiered discounts for business customers
- `GOVERNMENT_DISCOUNT`: Discount for government customers

### API Settings
```python
FRONTEND_URL = config('FRONTEND_URL')  # Used for redirects in payments, emails
BACKEND_URL = config('BACKEND_URL')    # Used for internal API communications
```

**API Parameters:**
- `FRONTEND_URL`: Full URL to the frontend application (used for redirect URLs)
- `BACKEND_URL`: Full URL to the backend API (for internal communications)

### Cloudinary Configuration
```python
import cloudinary
import cloudinary.uploader
import cloudinary.api

cloudinary.config(
    cloud_name=config('CLOUDINARY_CLOUD_NAME'),
    api_key=config('CLOUDINARY_API_KEY'),
    api_secret=config('CLOUDINARY_API_SECRET'),
    secure=True
)
```

### SSLCommerz Configuration
```python
SSLCOMMERZ_STORE_ID = config('SSLCOMMERZ_STORE_ID')
SSLCOMMERZ_STORE_PASS = config('SSLCOMMERZ_STORE_PASS')
SSLCOMMERZ_IS_SANDBOX = config('SSLCOMMERZ_IS_SANDBOX', cast=bool, default=True)
```

## Frontend Configuration

### Environment Variables
Frontend uses Vite's environment variable system with `VITE_` prefix:

```javascript
// API Configuration
VITE_API_BASE_URL: Base URL for backend API
VITE_SUPABASE_URL: Supabase project URL
VITE_SUPABASE_ANON_KEY: Supabase anonymous key

// Feature Flags
VITE_ENABLE_REALTIME: Enable/disable real-time features
VITE_DEBUG_MODE: Development debug features
```

### Build Configuration
- **Optimization**: Tree-shaking and code splitting enabled
- **Asset Handling**: Static asset optimization
- **Compression**: Gzip/Brotli compression enabled
- **Caching**: Long-term caching with content hashes

## Security Configuration

### Security Headers
```python
SECURE_SSL_REDIRECT = config('SECURE_SSL_REDIRECT', cast=bool, default=True)
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SECURE_SSL_HOST = config('SECURE_SSL_HOST', default=None)
SECURE_HSTS_SECONDS = config('SECURE_HSTS_SECONDS', cast=int, default=31536000)
SECURE_HSTS_INCLUDE_SUBDOMAINS = config('SECURE_HSTS_INCLUDE_SUBDOMAINS', cast=bool, default=True)
SECURE_HSTS_PRELOAD = config('SECURE_HSTS_PRELOAD', cast=bool, default=True)
SECURE_CONTENT_TYPE_NOSNIFF = config('SECURE_CONTENT_TYPE_NOSNIFF', cast=bool, default=True)
SECURE_BROWSER_XSS_FILTER = config('SECURE_BROWSER_XSS_FILTER', cast=bool, default=True)
SESSION_COOKIE_SECURE = config('SESSION_COOKIE_SECURE', cast=bool, default=True)
SESSION_COOKIE_HTTPONLY = config('SESSION_COOKIE_HTTPONLY', cast=bool, default=True)
SESSION_COOKIE_SAMESITE = config('SESSION_COOKIE_SAMESITE', default='Lax')
CSRF_COOKIE_SECURE = config('CSRF_COOKIE_SECURE', cast=bool, default=True)
CSRF_COOKIE_HTTPONLY = config('CSRF_COOKIE_HTTPONLY', cast=bool, default=False)
CSRF_COOKIE_SAMESITE = config('CSRF_COOKIE_SAMESITE', default='Lax')
```

### Allowed Hosts
```python
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=lambda v: [s.strip() for s in v.split(',')]) + [
    '127.0.0.1',
    'localhost',
    '.vercel.app',
]
```

## Logging Configuration

### Production Logging
```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
        'simple': {
            'format': '{levelname} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': config('LOG_FILE', default='debug.log'),
            'formatter': 'verbose',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
    },
    'root': {
        'handlers': ['console', 'file'],
        'level': config('LOG_LEVEL', default='INFO'),
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'level': config('DJANGO_LOG_LEVEL', default='INFO'),
            'propagate': False,
        },
        'django_auth_ldap': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
        },
    },
}
```

## Health Check Configuration

### Health Check Endpoints
- **Database Health**: Check database connectivity
- **Cache Health**: Check Redis connectivity
- **External Services**: Check third-party service connectivity
- **Application Health**: General application status

### Monitoring Configuration
- **Health Check URL**: `/health/` endpoint for deployment monitoring
- **Readiness Check**: Application readiness for traffic
- **Liveness Check**: Application liveness for container orchestration

## Management Commands

### Essential Commands
- `uv run manage.py migrate` - Apply database migrations
- `uv run manage.py createsuperuser` - Create admin user
- `uv run manage.py collectstatic` - Collect static files for production
- `uv run manage.py test` - Run application tests
- `uv run manage.py init_roles` - Initialize user roles and ensure all users have service profiles
- `uv run manage.py erase_post_db` - **DANGEROUS**: Erase entire PostgreSQL database and recreate schema (for development/reset purposes)

### Development Commands
- `uv run manage.py shell` - Open Django shell for debugging
- `uv run manage.py runserver` - Start development server
- `uv run manage.py makemigrations` - Create database migration files

## Performance Configuration

### Database Connection Pooling
- **Size**: Configurable connection pool size
- **Timeout**: Connection timeout settings
- **Recycling**: Connection recycling to prevent stale connections

### Caching Strategy
- **Page Cache**: Cache entire pages when appropriate
- **Fragment Cache**: Cache expensive template fragments
- **Query Cache**: Cache expensive database queries
- **Session Cache**: Efficient session storage

## Logging Configuration

### Application Logging
The application uses a comprehensive logging configuration with multiple handlers:

```python
LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "verbose": {
            "format": "{levelname} {asctime} {module} {process:d} {thread:d} {message}",
            "style": "{",
        },
        "simple": {
            "format": "{levelname} {message}",
            "style": "{",
        },
    },
    "handlers": {
        "file": {
            "level": "INFO",
            "class": "logging.FileHandler",
            "filename": "debug.log",
            "formatter": "verbose",
        },
        "security_file": {
            "level": "INFO",
            "class": "logging.FileHandler",
            "filename": "security.log",
            "formatter": "verbose",
        },
        "console": {
            "level": "INFO",
            "class": "logging.StreamHandler",
            "formatter": "simple",
        },
    },
    "loggers": {
        "django": {
            "handlers": ["file", "console"],
            "level": "INFO",
            "propagate": True,
        },
        "homeser.security": {
            "handlers": ["security_file", "console"],
            "level": "INFO",
            "propagate": False,
        },
        "homeser.exception_handler": {
            "handlers": ["file", "console"],
            "level": "ERROR",
            "propagate": False,
        },
    },
}
```

**Logging Components:**
- **File Handler**: Logs to `debug.log` with verbose formatting
- **Security Handler**: Logs to separate `security.log` file for security-related events
- **Console Handler**: Outputs to console for development/debugging
- **Django Logger**: Captures all Django framework events
- **Security Logger**: `homeser.security` specifically for security events
- **Exception Logger**: `homeser.exception_handler` for exception handling

## Related Documentation

- [See: deployment/vercel-setup.md]
- [See: deployment/environment-configs.md]
- [See: performance/caching-strategy.md]
- [See: references/glossary.md#configuration]