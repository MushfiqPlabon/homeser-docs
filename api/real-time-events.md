# Real-time Events API

This document describes the real-time update patterns and event flows in the HomeSer service marketplace using Supabase PostgreSQL change subscriptions.

## Overview

The real-time system provides live updates for critical events like booking status changes, payment confirmations, and availability updates. The system uses Supabase PostgreSQL change subscriptions to deliver updates efficiently while staying within free tier constraints.

## Real-time Architecture

### Supabase PostgreSQL Change Subscriptions
- Uses native PostgreSQL logical replication
- Subscribes to specific table changes (INSERT, UPDATE, DELETE)
- Filters subscriptions by user or resource ownership
- Maintains WebSocket connections for live updates

### Django-Supabase Synchronization
- Django models automatically synced to Supabase using Django signals
- Supabase client is lazy-loaded to prevent Django startup blocking
- Booking model: Synced via post_save/post_delete signals in `bookings/signals.py`
- Availability model: Synced via post_save signals in `bookings/signals.py`
- Sync operations: INSERT (new), UPDATE (changes), DELETE (removal)
- Real-time update propagation from Django models to frontend via Supabase
- Error handling: Try-catch blocks in signal handlers prevent sync failures from affecting core operations
- Notification integration: Status changes trigger notification system via `NotificationService.notify_booking_status_change()`
- Operation tracking: Pre-save signal tracks status changes for notification purposes

### Connection Management
- Automatic connection establishment when user logs in
- Reconnection logic with exponential backoff
- Connection status monitoring and reporting
- Graceful degradation when Supabase unavailable

## Event Categories

### Booking Events (Customer)
- **Channel**: `bookings-changes-{user_id}` (filtered for customer_id)
- **Events**: PostgreSQL change events (INSERT, UPDATE, DELETE) on bookings table where customer_id matches current user
- **Event Types**:
  - `*` (wildcard) - All changes to bookings for this customer

### Booking Events (Provider)
- **Channel**: `provider-bookings-changes-{provider_id}` (filtered for provider_id)
- **Events**: PostgreSQL change events (INSERT, UPDATE, DELETE) on bookings table where provider_id matches current provider
- **Event Types**:
  - `*` (wildcard) - All changes to bookings for this provider

### Notification Events
- **Channel**: `notifications-changes-{user_id}` (filtered for user_id)
- **Events**: PostgreSQL change events (INSERT, UPDATE, DELETE) on notifications table where user_id matches current user
- **Event Types**:
  - `*` (wildcard) - All changes to notifications for this user

### Service Status Events
- **Channel**: `services-changes` (unfiltered)
- **Events**: PostgreSQL change events (INSERT, UPDATE, DELETE) on services table
- **Event Types**:
  - `*` (wildcard) - All changes to services

## Event Payload Structure

### PostgreSQL Change Event Format
The Supabase real-time events provide raw PostgreSQL changes with the following structure:

```json
{
  "type": "INSERT" | "UPDATE" | "DELETE",
  "table": "bookings" | "notifications" | "services",
  "record": {
    /* Full database record as JSON */
  },
  "old_record": {
    /* Previous record state for UPDATE/DELETE events (null for INSERT) */
  },
  "columns": [
    /* Array of column definitions involved in the change */
  ]
}
```

### Booking Event Example (UPDATE)
```json
{
  "type": "UPDATE",
  "table": "bookings",
  "record": {
    "id": 123,
    "customer_id": 456,
    "provider_id": 789,
    "service_id": 101,
    "booking_date": "2023-12-25T10:00:00Z",
    "start_time": "2023-12-25T10:00:00Z",
    "end_time": "2023-12-25T12:00:00Z",
    "status": "confirmed",
    "updated_at": "2023-12-03T10:30:00Z",
    /* ... other booking fields */
  },
  "old_record": {
    "id": 123,
    "status": "pending",
    /* ... previous state */
  },
  "columns": [
    /* Column definitions */
  ]
}
```

## Subscription Management

### Customer Subscriptions
- Bookings: Subscribe to bookings where customer_id = current user
- Payments: Subscribe to payments linked to customer's bookings
- Notifications: Subscribe to notifications for current user

### Provider Subscriptions
- Bookings: Subscribe to bookings where provider_id = current provider
- Availability: Subscribe to availability updates for provider
- Notifications: Subscribe to notifications for current provider

### Admin Subscriptions
- All system events (where applicable permissions exist)
- System-wide notifications
- Critical system status updates

## Client-Side Implementation

### Subscription Hooks
The frontend uses custom React hooks to manage subscriptions:

#### useRealTimeBookings Hook
```javascript
const { bookings, loading, error } = useRealTimeBookings(userId);
```

#### useRealTimePayments Hook
```javascript
const { payments, loading, error } = useRealTimePayments(userId);
```

### Connection Status
- `CONNECTING` - Establishing connection
- `CONNECTED` - Active connection
- `RECONNECTING` - Attempting reconnection
- `DISCONNECTED` - Connection lost

## Error Handling

### Connection Errors
- **Network Issues**: Automatic reconnection with exponential backoff
- **Authentication Errors**: Refresh authentication and reconnect
- **Subscription Errors**: Retry subscription with backoff
- **Rate Limiting**: [See: security/security-measures.md#rate-limiting]

### Event Processing
- Validate event structure before processing
- Handle missing or malformed data gracefully
- Log errors without disrupting other events

## Performance Considerations

### Free Tier Optimization
- Selective subscriptions to reduce connection count
- Efficient filtering to minimize data transfer
- Connection pooling where possible
- Caching of static data to reduce real-time load

### Event Filtering
- Client-side filtering for non-critical updates
- Server-side filtering through PostgreSQL RLS
- Subscription-level filtering through channel parameters

### Data Efficiency
- Send only changed data when possible
- Use appropriate serialization
- Minimize payload size

## Security Considerations

### Data Access Control
- PostgreSQL Row Level Security (RLS) limits data exposure
- Channel subscriptions restricted to authorized data
- Authentication required for all subscriptions
- Event payload validation

### Connection Security
- Secure WebSocket connections (wss://)
- Token-based authentication for subscriptions
- Session validation before establishing connections
- Automatic logout on token expiration

## Related Documentation

- [See: api/booking-flow.md]
- [See: api/payment-flow.md]
- [See: security/security-measures.md]
- [See: performance/monitoring.md]
- [See: patterns/frontend-patterns.md#real-time]

## Implementation Notes

- Real-time connections use Supabase client library with custom reconnection logic
- Supabase client is lazy-loaded via `get_supabase_client()` to prevent Django startup blocking
- The system includes mock implementation for graceful degradation when Supabase unavailable
- Event payloads include versioning for future compatibility
- Connection monitoring helps identify and resolve connectivity issues quickly
- Signal-based synchronization: Uses Django post_save/post_delete/pre_save signals in `bookings/signals.py` to trigger real-time updates
- Database-level synchronization: The `homeser.realtime_sync.sync_to_supabase()` function handles actual Supabase table operations
- Error resilience: All sync operations wrapped in try-catch to prevent failures from impacting core functionality
- Status change tracking: Pre-save signal tracks booking status transitions for notification purposes
- Notification integration: Status changes automatically trigger notification services through the Django signal system