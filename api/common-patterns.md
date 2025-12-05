# Common API Patterns

This document describes the common API patterns used throughout the HomeSer service marketplace, ensuring consistency across all API endpoints.

## Overview

The HomeSer API follows consistent patterns across all endpoints to ensure predictability, maintainability, and ease of use for both frontend and any external integrations.

## Authentication Patterns

[See: security/security-measures.md#authentication-security]
[See: api/authentication-flow.md]

## API Endpoint Design

### RESTful Principles
- Use standard HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Use plural nouns for resource endpoints
- Use nested routes for relationships (e.g., `/api/bookings/{id}/cancel/`)

### URL Structure
```
/api/{version}/{resource}/{id}/{action}/
```
Example: `/api/v1/bookings/123/cancel/`

### Standard Response Format

#### Successful Responses
```json
{
  "success": true,
  "data": { /* resource data */ },
  "message": "Optional success message"
}
```

#### Error Responses
```json
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": { /* optional error details */ }
}
```

## Query Parameters

### Filtering
- Use consistent parameter names across endpoints
- Support multiple filters: `/api/services/?category=cleaning&location=123`
- Support date ranges: `/api/bookings/?start_date=2023-01-01&end_date=2023-01-31`

### Pagination
[See: deployment/common-configuration.md#pagination-configuration] for pagination settings and limits.

### Search and Sorting
- Search: `/api/services/services/?search=keyword` (for services) or `/api/services/intelligent-search/` (for intelligent ranking)
- Sorting: `/api/services/services/?ordering=-created_at,rating`
- Multi-field search support

## Request Body Patterns

### Standard Resource Creation
```json
{
  "name": "Resource name",
  "description": "Resource description",
  "additional_field": "value"
}
```

### Bulk Operations
Support for bulk operations where appropriate:
- Bulk creation: POST with array of resources
- Bulk updates: PATCH with array of resources

### Partial Updates
Use PATCH method for partial resource updates:
```json
{
  "optional_field": "new_value"
}
```

## Error Handling Patterns

### HTTP Status Codes
- 200: Successful GET, PUT, PATCH
- 201: Successful POST
- 204: Successful DELETE
- 400: Bad Request (validation errors)
- 401: Unauthorized
- 403: Forbidden (insufficient permissions)
- 404: Not Found
- 422: Unprocessable Entity (business logic validation)
- 429: Too Many Requests (rate limiting)
- 500: Internal Server Error

### Validation Error Format
```json
{
  "error": "Validation failed",
  "code": "VALIDATION_ERROR",
  "details": {
    "field_name": [
      "Error message 1",
      "Error message 2"
    ]
  }
}
```

## Caching Strategies

### Cache Headers
- Set appropriate cache-control headers for different endpoints
- Use ETags for conditional requests
- Implement cache invalidation for related resources

### Server-Side Caching
- Cache expensive operations (e.g., provider ranking)
- Use Redis with appropriate TTL for different data types
- Invalidate cache when related data is modified

## Real-time Integration

### WebSocket Connection
- Establish connection through Supabase real-time features
- Handle connection status and reconnection logic
- Subscribe to relevant channels based on user context

### Event Naming Convention
- Use consistent naming: `{resource}.{action}` (e.g., `booking.updated`)
- Include relevant data in event payload
- Handle errors gracefully during real-time updates

## Analytics Integration

### Customer Behavior Tracking
- **Endpoint**: `/api/analytics/track/`
- **Method**: POST
- **Authentication**: Optional (tracks both authenticated and anonymous users)
- **Request Body**:
```json
{
  "session_id": "unique_session_identifier",
  "event_type": "page_view",
  "data": {
    "custom_field": "custom_value"
  },
  "page_url": "/page/path"
}
```

### Tracking Patterns
- Session-based tracking for anonymous users
- User-based tracking for authenticated users
- Event type categorization for analytics
- JSON data field for flexible tracking information

## Real-time and Notification Patterns

### Django-Supabase Synchronization
- **Mechanism**: Django signals automatically sync model changes to Supabase
- **Triggers**: post_save, post_delete signals for real-time updates
- **Models Synced**: Booking, Availability models currently
- **Operations**: INSERT, UPDATE, DELETE operations synchronized
- **Purpose**: Enable real-time updates between Django backend and React frontend

### Notification Service
- **Service**: `notifications.services.NotificationService`
- **Features**:
  - In-app notifications via Notification model
  - Email notifications via SMTP
  - Booking status change notifications
  - Payment completion notifications
- **Integration**: Called from Django signals on model changes

## Third-Party Integrations

### Appointment Scheduling
- **Package**: django-appointment for advanced scheduling
- **Integration**: `bookings.appointment_integration` module
- **Features**: Creates appointments from bookings, syncs booking data to external scheduling system
- **Usage**: Automatic synchronization when bookings are created/updated

## Error Handling Patterns

### Customer-Friendly Error Messages
- **Custom Exception Handler**: `homeser.exception_handler.custom_exception_handler`
- **Purpose**: Transform technical errors into customer-friendly messages
- **Features**:
  - Maps technical errors to user-understandable messages
  - Preserves original error details for debugging
  - Handles common error scenarios (permissions, validation, authentication, etc.)
  - Standardized error response format with status_code and error_details

## Notification Patterns

### Notification Management
- **List Notifications**: `/api/notifications/` (GET) - Get user's notifications
- **Mark as Read**: `/api/notifications/{id}/mark_read/` (POST) - Mark specific notification as read
- **Mark All Read**: `/api/notifications/mark_all_read/` (POST) - Mark all notifications as read
- **Unread Count**: `/api/notifications/unread_count/` (GET) - Get count of unread notifications

## Performance Patterns

### Optimistic Updates
- Update UI immediately on user action
- Revert on error response
- Invalidate cache on success

### Lazy Loading
- Support pagination for large datasets
- Implement infinite scrolling where appropriate
- Preload related data when possible

### Data Serialization Optimization
- Use specific serializers for different views
- Implement select_related/prefetch_related for relationships
- Limit fields returned when appropriate

## Security Patterns

### Rate Limiting
[See: security/security-measures.md#rate-limiting]

### Input Validation
- Server-side validation for all inputs
- Sanitize all user inputs
- Check permissions before processing

### Data Access Control
- Always filter data based on user permissions
- Use role-based access control
- Validate ownership when necessary

## Versioning Strategy

### API Versioning
- Include version in URL: `/api/v1/`
- Maintain backward compatibility when possible
- Document breaking changes clearly

## Common Headers

### Request Headers
- `Authorization`: JWT token for authenticated requests
- `Content-Type`: `application/json` for JSON requests
- `Accept`: `application/json` for JSON responses

### Response Headers
- `Content-Type`: `application/json`
- `Cache-Control`: Appropriate caching directives
- `ETag`: For conditional requests

## Health Check Endpoints

### Health Check
- **Method**: GET
- **Endpoint**: `/health/`
- **Purpose**: Basic health check for deployment monitoring
- **Response**:
```json
{
  "status": "healthy"
}
```
- **Authentication**: Not required
- **Authorization**: Public access
- **Use Case**: Health checks for deployment platforms, load balancers, and monitoring tools

## Related Documentation

- [See: api/booking-flow.md]
- [See: api/payment-flow.md]
- [See: security/security-measures.md]
- [See: patterns/backend-patterns.md]