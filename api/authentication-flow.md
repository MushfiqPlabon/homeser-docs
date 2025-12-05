# Authentication Flow API

This document describes the authentication API endpoints and request/response flows.

## Overview

[See: security/authentication-system.md] for architecture and dual authentication system details.

## User Types and Roles
- **Customer**: Standard user for booking services
- **Provider**: Service provider with additional service management capabilities
- **Admin**: Full system access for management and oversight

## Authentication Flow

[See: security/authentication-system.md#authentication-flows] for detailed flow descriptions.

## API Endpoints

### Login
- **Method**: POST
- **Endpoint**: `/api/accounts/login/`
- **Request**:
```json
{
  "email": "user@example.com",
  "password": "secure_password"
}
```
- **Response**:
```json
{
  "refresh": "jwt_refresh_token",
  "access": "jwt_access_token"
}
```
- **Authentication**: Not required
- **Authorization**: Public access

### Email Verification Request
- **Method**: POST
- **Endpoint**: `/api/accounts/send-verification/`
- **Request**: No request body required
- **Response**: Success message
- **Authentication**: Required
- **Authorization**: Authenticated user only

### Email Verification (via link)
- **Method**: GET
- **Endpoint**: `/api/accounts/verify-email/{uidb64}/{token}/`
- **Request**: URL parameters only (uidb64, token)
- **Response**: Verification status
- **Authentication**: Not required (token-based verification)
- **Authorization**: Public access via one-time token

### Registration
- **Method**: POST
- **Endpoint**: `/api/accounts/register/`
- **Request**:
```json
{
  "email": "user@example.com",
  "password": "secure_password",
  "first_name": "John",
  "last_name": "Doe",
  "user_type": "customer"
}
```
- **Response**: User profile with initial tokens
- **Authentication**: Not required
- **Authorization**: Public access

### Token Refresh
- **Method**: POST
- **Endpoint**: `/api/accounts/token/refresh/`
- **Request**:
```json
{
  "refresh": "jwt_refresh_token"
}
```
- **Response**:
```json
{
  "access": "new_jwt_access_token"
}
```
- **Authentication**: Requires valid refresh token
- **Authorization**: Token-based access

### Profile Management
- **Method**: GET
- **Endpoint**: `/api/accounts/profile/`
- **Request**: No request body
- **Response**: Current user profile information
- **Authentication**: Required
- **Authorization**: Own profile access

### Profile Update
- **Method**: PATCH
- **Endpoint**: `/api/accounts/profile/`
- **Request**: Partial profile updates
- **Response**: Updated user profile information
- **Authentication**: Required
- **Authorization**: Own profile access

### Logout
- **Method**: Client-side operation
- **Implementation**: Clear authentication tokens from client storage
- **Token Management**: Refresh tokens are automatically blacklisted on rotation (server-side)
- **Action**: Remove tokens from localStorage/sessionStorage and clear user state in frontend

## Dual Authentication Implementation

### Supabase Integration
- Conditional initialization based on credentials availability
- Graceful fallback when Supabase unavailable
- Session synchronization between systems
- Real-time authentication status updates

### Django Authentication
- JWT-based token system with configurable lifetimes
- Role-based access control with custom permissions
- Django Guardian for object-level permissions
- Email verification and account activation

## Security Measures

### Password Security
- Django's built-in password validation
- Configurable minimum password requirements
- Automatic password hashing (PBKDF2 by default)
- Rate limiting on authentication endpoints

### Token Security
- Secure signing with Django SECRET_KEY
- Configurable token lifetimes
- Refresh token rotation and blacklisting
- Automatic token refresh in frontend

### Session Management
- Concurrent session limits
- Session timeout configuration
- Automatic logout on password change
- Session tracking and management

## Real-time Authentication

### Session Events
- Real-time updates for authentication status changes
- Session expiration notifications
- Concurrent session detection
- Automatic reconnection handling

### Channel Subscriptions
- User-specific authentication event channels
- Session status change notifications
- Token expiration warnings
- Connection status monitoring

## Error Handling

### 400 Bad Request
- **Code**: `AUTH_VALIDATION_ERROR`
- **Message**: Authentication request validation failed

### 401 Unauthorized
- **Code**: `AUTH_INVALID_CREDENTIALS`
- **Message**: Invalid email or password

### 403 Forbidden
- **Code**: `AUTH_ACCOUNT_INACTIVE`
- **Message**: Account is inactive or requires verification

### 429 Too Many Requests
- **Code**: `AUTH_RATE_LIMITED`
- **Message**: Too many login attempts, try again later

## Performance Considerations

- Caching of user profiles and permissions
- Optimized database queries for authentication
- Asynchronous token validation where appropriate
- Efficient session management to reduce database load

## Related Documentation

- [See: security/authentication-system.md]
- [See: security/common-measures.md]
- [See: api/common-patterns.md]
- [See: patterns/backend-patterns.md#authentication]
- [See: references/glossary.md#authentication]

## Implementation Notes

- The dual authentication system is managed in `authStore.jsx` in the frontend
- Authentication decorators (`@require_role`, `@require_booking_access`) provide fine-grained access control
- The system gracefully handles Supabase unavailability with Django-only fallback
- Password reset and email verification are implemented with both systems