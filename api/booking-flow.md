# Booking Flow API

This document describes the API flow for the booking system in the HomeSer service marketplace.

## Overview

The booking flow enables customers to search for services, check provider availability, book time slots, and manage their bookings. The process involves multiple API calls with real-time updates for status changes. The system integrates with django-appointment for advanced scheduling features.

## Flow Overview

1. Customer searches for services with location and time preferences
2. Intelligent provider ranking returns sorted results
3. Customer selects service and checks provider availability
4. Customer books specific time slot through availability calendar
5. Booking is created and payment is processed
6. Real-time updates notify of booking status changes
7. Service is provided and customer can leave review

## API Endpoints

### Service Search with Intelligent Ranking
- **Method**: GET
- **Endpoint**: `/api/services/intelligent-search/`
- **Request**: Query parameters for location, category, date
- **Response**: Ranked list of services with availability
- **Authentication**: Not required for basic search
- **Authorization**: Public access

### Get Service Details
- **Method**: GET
- **Endpoint**: `/api/services/services/{id}/`
- **Request**: Service ID in URL path
- **Response**: Service details including provider info
- **Authentication**: Not required
- **Authorization**: Public access

### Check Provider Availability
- **Method**: GET
- **Endpoint**: `/api/bookings/availabilities/`
- **Request**: Provider ID as query parameter (`?provider={provider_id}`)
- **Response**: Available time slots for the provider, filtered by provider ID
- **Authentication**: Required
- **Authorization**: Customer role or above

### Create Booking
- **Method**: POST
- **Endpoint**: `/api/bookings/bookings/`
- **Request**:
```json
{
  "service": 123,
  "provider": 456,
  "booking_date": "2025-12-25T10:00:00Z",
  "start_time": "2025-12-25T10:00:00Z",
  "duration": 120,
  "customer_notes": "Additional instructions"
}
```
- **Note**: `end_time` is optional and set when service completes. Use `duration` (in minutes) for scheduling.
- **Response**: Created booking object with status and payment info
- **Authentication**: Required
- **Authorization**: Customer role

### Get Booking Details
- **Method**: GET
- **Endpoint**: `/api/bookings/bookings/{id}/`
- **Request**: Booking ID in URL path
- **Response**: Complete booking details
- **Authentication**: Required
- **Authorization**: Customer or provider associated with booking

### Update Booking
- **Method**: PATCH
- **Endpoint**: `/api/bookings/bookings/{id}/`
- **Request**: Partial booking update
- **Response**: Updated booking object
- **Authentication**: Required
- **Authorization**: Customer or provider associated with booking

### Cancel Booking
- **Method**: POST
- **Endpoint**: `/api/bookings/bookings/{id}/cancel/`
- **Request**: Cancellation reason (optional)
- **Response**: Updated booking with cancelled status
- **Authentication**: Required
- **Authorization**: Customer associated with booking

### Booking Status Changes
- **Method**: POST
- **Endpoint**: `/api/bookings/bookings/{id}/confirm/` (for confirmation)
- **Method**: POST
- **Endpoint**: `/api/bookings/bookings/{id}/cancel/` (for cancellation)
- **Response**: Updated booking object
- **Authentication**: Required
- **Authorization**: Provider or admin role for confirmation, customer for cancellation

## Real-time Updates

### Booking Status Updates
- Channel: `bookings:{booking_id}`
- Event: `booking.status_updated`
- Payload: Updated booking status and details

### Provider Availability Changes
- Channel: `providers:{provider_id}`
- Event: `availability.updated`
- Payload: Updated availability information

### Notification Updates
- Channel: `notifications:{user_id}`
- Event: `notification.created`
- Payload: New notification details

## Error Handling

### 400 Bad Request
- **Code**: `VALIDATION_ERROR`
- **Message**: Booking validation failed (time conflicts, invalid dates, etc.)

### 409 Conflict
- **Code**: `BOOKING_CONFLICT`
- **Message**: Time slot already booked or provider unavailable

### 422 Unprocessable Entity
- **Code**: `BOOKING_NOT_MODIFIABLE`
- **Message**: Booking cannot be modified in current state

## Security Considerations

- Validate that customer owns the booking before allowing modifications
- Check provider availability before creating booking
- Verify that requested time slots are valid and in the future
- Implement rate limiting on booking creation endpoints
- Use optimistic locking for concurrent booking attempts

## Performance Considerations

- Cache provider availability for the day to reduce database queries
- Use intelligent provider ranking with O(1) cached calculations
- Implement pagination for booking history
- Cache service availability calendar data

## Related Documentation

- [See: api/common-patterns.md]
- [See: security/security-measures.md]
- [See: performance/caching-strategy.md]
- [See: api/real-time-events.md]
- [See: data/models.md]

## Implementation Notes

- The booking system includes sophisticated validation through `BookingValidationService`
- Availability checking is performed by `availability_checker.py` 
- Time slot management ensures proper conflict prevention
- The system supports multiple booking statuses: pending, confirmed, in_progress, completed, cancelled, no_show, rescheduled