# Common Security Measures

This document describes the common security measures implemented across the HomeSer service marketplace, following security best practices and ensuring protection against common vulnerabilities.

## Overview

The HomeSer security framework implements multiple layers of security controls across all components of the application to protect data, maintain user privacy, and ensure the integrity of the system.

## Authentication Security

### JWT Token Security
- **Token Signing**: HS256 algorithm with Django SECRET_KEY
- **Token Rotation**: Refresh tokens are single-use and rotated
- **Blacklisting**: Refresh tokens are blacklisted after use
- **Lifetime Configuration**: Configurable token lifetimes via environment variables
- **Secure Storage**: Tokens stored securely in frontend (with considerations for localStorage vulnerabilities)

### Multi-System Authentication
- **Primary**: Django JWT for API authentication
- **Secondary**: Supabase for real-time features
- **Fallback**: Django-only authentication when Supabase unavailable
- **Synchronization**: Consistent authentication state across systems

## Authorization Security

### Role-Based Access Control (RBAC)
- **User Types**: Customer, Provider, Admin roles
- **Granular Permissions**: Custom permission classes and functions for specific actions
- **Custom Permissions**:
  - Service permissions: `view_own_services`, `add_own_services`, `change_own_services`, `delete_own_services`
  - Booking permissions: `view_provided_bookings`, `change_provided_bookings`, `view_own_bookings`, `change_own_bookings`
- **Object-Level Permissions**: Django Guardian for fine-grained access control
  - Per-object permission checks: `user.has_perm('bookings.view_booking', booking)`
  - Service-specific object permissions: `user.has_perm('services.change_service', service)`
  - Dynamic permission evaluation based on object ownership
- **Authorization Functions**:
  - `can_manage_service(user, service)` - Check if user can manage a specific service
  - `can_manage_booking(user, booking)` - Check if user can manage a specific booking
- **Decorator Pattern**: Custom decorators for role-based access control
- **Permission Assignment**: Role-based permission assignment via `assign_role_permissions()` function
- **Permission Inheritance**: Advanced role + object permission combination for complex scenarios

### Access Validation
- **Ownership Checks**: Verify user ownership of resources
- **Status-Based Access**: Different access based on booking/payment status
- **Time-Based Restrictions**: Access limited by booking times where applicable

## Input Validation and Sanitization

### API Input Validation
- **Django REST Framework Serializers**: Built-in validation for all inputs
- **Field Validation**: Comprehensive field validation rules
- **Custom Validators**: Application-specific validation logic
- **Rate Limiting**: Prevent abuse through throttling

### Business Rule Validation
- **Provider Availability Checks**: Validates provider availability before booking creation
- **Service Status Validation**: Ensures only active services can be booked
- **Booking Modification Constraints**: Prevents modification of completed/cancelled bookings
- **Booking Status Transitions**: Validates appropriate status change flows (e.g., only pending bookings can be confirmed)
- **Authorization Validation**: Validates user permissions for specific actions (e.g., only customer/provider can confirm booking)
- **Temporal Validation**: Validates booking dates are not in the past

### Data Sanitization
- **Output Encoding**: Prevent XSS through proper output encoding
- **SQL Injection Prevention**: Django ORM prevents SQL injection
- **NoSQL Injection Prevention**: Proper query construction in all components

## Data Protection

### Data Classification
- **Public Data**: Service listings, provider information (public view)
- **Private Data**: User profiles, bookings, payment information
- **Sensitive Data**: Payment details, authentication tokens
- **Restricted Data**: Administrative data and system logs

### Encryption
- **At Rest**: Database encryption through Supabase hosting
- **In Transit**: HTTPS/TLS for all communication
- **Token Encryption**: JWT tokens with secure signing
- **File Encryption**: Image storage with Cloudinary security

## API Security

### Rate Limiting
- **Anonymous Users**: 100 requests per hour (via DRF throttling)
- **Authenticated Users**: 1000 requests per hour (via DRF throttling)
- **Special Endpoints**: More restrictive limits on sensitive endpoints (via DRF throttling)
- **Custom Rules**: Endpoint-specific rate limiting through DRF throttling
- **Planned Enhancement**: `django-ratelimit` package installed but not yet implemented on authentication endpoints to prevent brute force attacks (addresses security vulnerability noted in vulnerabilities.md)

### CORS Configuration
- **Frontend Domain**: Only allow specified frontend domains
- **HTTP Methods**: Restrict allowed HTTP methods
- **Headers**: Control allowed headers and credentials

## Database Security

### Row-Level Security (RLS)
- **Supabase Implementation**: PostgreSQL RLS policies
- **User Isolation**: Users can only access their own data
- **Role-Based Access**: Different access levels based on user roles
- **Performance Indexes**: Optimized for RLS queries

### Application-Level Queryset Filtering
- **Role-Based Queryset Mixin**: `RoleBasedQuerysetMixin` provides standardized filtering
- **Admin Access**: Administrators get unrestricted access to all records
- **User Access**: Regular users only see records matching their user ID
- **Field Flexibility**: Configurable user field mapping (e.g., "user", "customer", "provider")
- **Implementation**: Used in ViewSets via `filter_queryset_by_role()` method throughout the application

### Query Security
- **ORM Usage**: Django ORM to prevent injection attacks
- **Parameterized Queries**: All queries use parameterized approach
- **Select/Prefetch Related**: Optimize queries to prevent N+1 issues

## Communication Security

### HTTPS Enforcement
- **Redirect Configuration**: SSL redirect configurable via environment (default: False/non-production, should be True in production)
- **Secure Headers**: Implement security headers for all responses
- **Cookie Security**: Configurable secure cookies (default: False in development, should be True in production)

### Security Headers Configuration
- **X-Frame-Options**: DENY - Prevent clickjacking attacks
- **X-Content-Type-Options**: nosniff - Prevent MIME-type confusion
- **X-XSS-Protection**: 1; mode=block - Browser XSS protection
- **HSTS Configuration**:
  - `SECURE_HSTS_SECONDS`: 31536000 seconds (1 year) - HTTP Strict Transport Security max age
  - `SECURE_HSTS_INCLUDE_SUBDOMAINS`: True - Apply HSTS to all subdomains
  - `SECURE_HSTS_PRELOAD`: True - Allow inclusion in browser HSTS preload lists
- **Cookie Security**:
  - `SESSION_COOKIE_SECURE`: Configurable (default: False, enable in production) - Secure session cookies
  - `CSRF_COOKIE_SECURE`: Configurable (default: False, enable in production) - Secure CSRF cookies

## Session Management

### Token Management
- **Short-Lived Access Tokens**: Minimize exposure window
- **Longer-Lived Refresh Tokens**: Secure rotation mechanism
- **Token Blacklisting**: Prevent replay attacks
- **Automatic Refresh**: Seamless token renewal for users

### Session Security
- **Concurrent Sessions**: Controls on multiple simultaneous sessions
- **Session Timeout**: Configurable timeout settings
- **Session Hijacking Prevention**: Secure session tokens
- **Logout Procedures**: Complete session termination

## Error Handling Security

### Information Disclosure Prevention
- **Generic Error Messages**: Avoid system information in errors
- **Log Redaction**: Sensitive data not logged
- **Customer-Friendly Errors**: Clear but secure error messages
- **Technical Error Logging**: Separate secure logging for developers

### Exception Handling
- **Centralized Handler**: `homeser/exception_handler.py` for all exceptions
- **Consistent Format**: Standardized error response format
- **Security Audits**: Regular review of error handling

## Secure Configuration

### Environment Management
- **Credential Storage**: All credentials from environment variables
- **Validation**: Environment variable validation at startup
- **Default Safety**: Safe defaults when environment variables missing

### Security Settings
- **Debug Mode**: Never enabled in production
- **Allowed Hosts**: Properly configured for production
- **Security Middleware**: All security middleware enabled

## Monitoring and Logging

### Security Monitoring
- **Authentication Events**: Log all login/logout attempts
- **Authorization Failures**: Track access violations
- **Suspicious Activity**: Monitor for unusual patterns
- **Audit Trails**: Maintain complete audit logs

### Log Security
- **PII Exclusion**: Personal data not stored in logs
- **Secure Storage**: Encrypted log storage
- **Access Controls**: Limited access to logs
- **Retention Policies**: Appropriate retention and destruction

## Vulnerability Management

### Regular Security Assessments
- **Code Reviews**: Security-focused code reviews
- **Dependency Scanning**: Regular scanning of dependencies
- **Vulnerability Testing**: Periodic security testing
- **Penetration Testing**: Regular security assessments

### Security Updates
- **Dependency Updates**: Regular updates of all dependencies
- **Framework Updates**: Keep Django and libraries current
- **Security Patches**: Apply security patches promptly

## Related Documentation

- [See: security/authentication-system.md]
- [See: api/common-patterns.md#security-patterns]
- [See: deployment/environment-configs.md]
- [See: references/glossary.md#security]