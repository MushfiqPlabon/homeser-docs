# Environment Configuration

This document describes all the environment variables required for the HomeSer service marketplace, organized by category. These variables must be configured in both development and production environments.

## Overview

HomeSer uses environment variables for all configuration to ensure security best practices and deployment flexibility. The application validates required variables at startup and provides safe defaults where appropriate.

## Required Variables

### Django Core Settings
```bash
SECRET_KEY=your-very-long-secret-key-here
DEBUG=False  # True for development, False for production
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com,.yourdomain.com
```

### Database Configuration
```bash
# Primary method (recommended):
DATABASE_URL=postgresql://user:pass@host:6543/database

# Fallback method (if DATABASE_URL not set):
DB_NAME=your_database_name
DB_USER=your_database_user
DB_PASSWORD=your_database_password
DB_HOST=your_postgres_host.supabase.co
DB_PORT=6543
```
[See: deployment/common-configuration.md#database-configuration] for detailed DATABASE_URL format and connection pooling settings.

## Authentication & Security

### JWT Configuration
```bash
JWT_ACCESS_TOKEN_LIFETIME_MINUTES=15  # Access token lifetime in minutes
JWT_REFRESH_TOKEN_LIFETIME_DAYS=7     # Refresh token lifetime in days
```

### Supabase Authentication
```bash
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-anon-public-key
SUPABASE_SERVICE_KEY=your-service-role-key
```

## Storage & CDN

### Cloudinary Configuration
```bash
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret
```

### Redis Cache Configuration
```bash
REDIS_URL=redis://upstash-redis-url-goes-here
```

## Payment Processing

### SSLCommerz Configuration
```bash
SSLCOMMERZ_STORE_ID=your_sslcommerz_store_id
SSLCOMMERZ_STORE_PASS=your_sslcommerz_store_password
SSLCOMMERZ_IS_SANDBOX=True  # Set to False for production
```

## Email Configuration
```bash
EMAIL_HOST=smtp.yourprovider.com
EMAIL_PORT=587
EMAIL_HOST_USER=your_smtp_username
EMAIL_HOST_PASSWORD=your_smtp_password
EMAIL_USE_TLS=True
DEFAULT_FROM_EMAIL=noreply@yourdomain.com
```
[See: deployment/common-configuration.md#email-configuration] for detailed email configuration parameters.

## CORS & Frontend Integration
```bash
CORS_ALLOWED_ORIGINS=https://your-frontend-domain.com,http://localhost:3000
CSRF_TRUSTED_ORIGINS=https://your-backend-domain.com,http://localhost:8000
FRONTEND_URL=https://your-frontend-domain.com
BACKEND_URL=https://your-backend-domain.com
```

## Business Configuration
```bash
TAX_RATE=0.15  # Tax rate as decimal (15% = 0.15)
SERVICE_FEE_RATE=0.05  # Service fee rate as decimal (5% = 0.05)
EXPRESS_FEE=10.00  # Additional fee for express services
PRIORITY_SUPPORT_COST=50.00  # Cost for priority support
WARRANTY_COST_PER_MONTH=25.00  # Warranty cost per month
BUSINESS_SMALL_DISCOUNT=0.05  # Small business discount (5%)
BUSINESS_MEDIUM_DISCOUNT=0.10  # Medium business discount (10%)
BUSINESS_LARGE_DISCOUNT=0.15  # Large business discount (15%)
GOVERNMENT_DISCOUNT=0.10  # Government discount (10%)
```

## Security Headers (Production)
```bash
SECURE_SSL_REDIRECT=True  # Enable in production
SECURE_HSTS_SECONDS=31536000  # 1 year HSTS
SECURE_HSTS_INCLUDE_SUBDOMAINS=True
SECURE_HSTS_PRELOAD=True
SESSION_COOKIE_SECURE=True  # Enable in production (with SSL)
CSRF_COOKIE_SECURE=True  # Enable in production (with SSL)
```

## Validation and Monitoring

### Environment Variable Validation
The system validates all required environment variables at startup:
- Missing required variables cause application startup failure
- Validation prevents deployment with incomplete configuration
- Safe defaults used for optional variables that are missing

### Validation Script
Use the validation script to check all environment variables:
```bash
python validate_env.py
```

This script checks for required variables and provides detailed feedback on missing or invalid configurations.

## Development Environment

### Development Variables
```bash
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
DB_HOST=localhost
DB_PORT=5432
FRONTEND_URL=http://localhost:3000
BACKEND_URL=http://localhost:8000
SSLCOMMERZ_IS_SANDBOX=True
```

### Local Development
For local development, create a `.env` file with all required variables:
```
SECRET_KEY=your-local-secret-key-for-development
DEBUG=True
# ... other variables as needed
```

## Production Environment

### Production Variables
- **DEBUG**: Must be `False`
- **ALLOWED_HOSTS**: Must include production domain
- **SECURE_*** settings: Should be enabled
- **SSLCommerz**: SANDBOX should be `False`
- **Email Settings**: Configure production SMTP

## Container/Docker Environment

When using containerized deployments, ensure environment variables are properly passed to containers:
```dockerfile
ENV SECRET_KEY=${SECRET_KEY}
ENV DATABASE_URL=${DATABASE_URL}
# etc.
```

## Environment Variable Management Best Practices

### Security
- Never commit environment variables to source code
- Use `.env.example` for variable templates
- Rotate credentials regularly
- Store variables in secure configuration management systems

### Validation
- Test variables in staging before production
- Validate required variables at application startup
- Use validation scripts to check configuration
- Monitor for configuration drift

### Documentation
- Document all environment variables in this file
- Update documentation when adding new variables
- Specify required vs optional variables clearly
- Include default values and usage notes

## Environment-Specific Configurations

### Development vs Production Differences
| Setting | Development | Production |
|---------|-------------|------------|
| DEBUG | True | False |
| SECURE_SSL_REDIRECT | False | True |
| SESSION_COOKIE_SECURE | False | True |
| CSRF_COOKIE_SECURE | False | True |
| SUPABASE_IS_SANDBOX | True | False |

## Troubleshooting

### Common Environment Issues
- **Missing SECRET_KEY**: Application won't start without it
- **Invalid Database Credentials**: Connection failures
- **Wrong ALLOWED_HOSTS**: HTTP 400 errors
- **Missing Payment Credentials**: Payment features not available
- **Incorrect CORS Settings**: Frontend-backend communication issues

### Configuration Validation
Always run environment validation after configuring:
```bash
python validate_env.py  # Should show no missing required variables
```

## Related Documentation

- [See: deployment/common-configuration.md]
- [See: deployment/vercel-setup.md]
- [See: security/vulnerabilities.md]
- [See: security/common-measures.md]
- [See: references/glossary.md#environment-variables]