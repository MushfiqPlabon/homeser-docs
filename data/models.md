# Data Models

This document describes the core data models in the HomeSer service marketplace, including their structure, relationships, and business logic implementation.

## Overview

The HomeSer data models are organized by application domain (accounts, services, bookings, payments) and follow consistent patterns for structure, validation, and relationships. This document provides detailed information about each model and its role in the system.

## Accounts Models

### ServiceProfile Model
The ServiceProfile model extends user functionality for marketplace roles.

**Fields:**
- `user`: One-to-one relationship with Django's built-in User model (primary key)
- `user_type`: Choice field (customer, provider, admin) - default: "customer"
- `phone_number`: Contact information (optional)
- `address`: Physical address information (optional)
- `bio`: Provider description (for providers), max 500 characters
- `is_verified`: Verification status for providers
- `email_verified`: Email verification status
- `latitude`, `longitude`: Geographic coordinates for intelligent ranking
- `jobs_count`: Number of completed jobs (for providers)
- `created_at`, `updated_at`: Timestamps (auto-populated)

**Business Logic:**
- Implements role-based functionality (is_customer(), is_provider(), is_admin())
- Manages provider location data for intelligent ranking
- Tracks provider performance metrics
- Enforces verification requirements
- Authorization methods:
  - `can_manage_service(service)` - Check if user can manage a specific service
  - `can_manage_booking(booking)` - Check if user can manage a specific booking

## Services Models

### ServiceCategory Model
Categorization of services offered on the platform.

**Fields:**
- `name`: Category name (max 100, unique)
- `description`: Category description (optional)
- `icon`: CloudinaryField for category icon (optional)
- `is_active`: Category active status - default: True
- `created_at`, `updated_at`: Timestamps (auto-populated)

**Business Logic:**
- Enforces unique category names
- Supports ordering by name

### Service Model
Core service entity representing offerings by providers.

**Fields:**
- `name`: Service name (max 200 characters)
- `description`: Detailed service description (max 1000 characters)
- `category`: Foreign key to ServiceCategory
- `provider`: Foreign key to Django User (provider)
- `price`: Service price (DecimalField with max 10 digits, 2 decimal places)
- `duration`: Estimated service duration in minutes
- `image`: CloudinaryField for service image (optional)
- `is_active`: Service availability - default: True
- `is_featured`: Featured service status - default: False
- `is_verified`: Service verification status - default: False
- `booking_policy`: Booking terms and conditions (optional)
- `cancellation_policy`: Cancellation terms (optional)
- `service_area`: Areas where service is provided (optional)
- `max_travel_distance`: Maximum distance provider will travel (km) - default: 0
- `required_skills`: Required skills for this service (optional)
- `certifications`: Required certifications (optional)
- `created_at`, `updated_at`: Timestamps (auto-populated)

**Business Logic:**
- Manages service pricing and availability
- Implements service verification workflow
- Enforces provider relationship

### ServiceImage Model
Multiple images per service with primary flag.

**Fields:**
- `service`: Foreign key to Service
- `image`: CloudinaryField for image storage
- `alt_text`: Image description (max 200 characters)
- `is_primary`: Primary image flag - default: False
- `created_at`: Timestamp (nullable, for backward compatibility)

**Business Logic:**
- Manages primary image designation
- Handles Cloudinary integration
- Supports ordering by primary status and creation date

### ServiceReview Model
Customer reviews for services.

**Fields:**
- `service`: Foreign key to Service
- `customer`: Foreign key to Django User (customer)
- `rating`: Integer field (1-5 stars)
- `review_text`: Review content (optional)
- `is_verified_booking`: Verified booking review flag - default: False
- `created_at`, `updated_at`: Timestamps (auto-populated)

**Business Logic:**
- Prevents duplicate reviews per customer-service pair
- Supports ordering by creation date
- Enforces rating to be between 1-5 stars

## Bookings Models

### Booking Model
Core booking entity for time-slot reservations.

**Fields:**
- `customer`: Foreign key to Django User (customer)
- `provider`: Foreign key to Django User (provider)
- `service`: Foreign key to Service
- `booking_date`: Date/time of the booking (DateTimeField)
- `start_time`: Booking start time (DateTimeField, optional)
- `end_time`: Booking end time (DateTimeField, optional)
- `duration`: Duration in minutes - default: 60
- `status`: Choice field (pending, confirmed, in_progress, completed, cancelled, no_show, rescheduled) - default: "pending"
- `priority`: Choice field (normal, express, priority) - default: "normal"
- `special_instructions`: Special requirements from customer (optional)
- `customer_address`: Address for on-site services - default: ""
- `service_location_type`: Choice field (onsite, remote, hybrid) - default: "onsite"
- `customer_rating`: Customer's rating of service (1-5) (optional)
- `customer_review`: Customer's review text (optional)
- `provider_rating`: Provider's rating of customer (1-5) (optional)
- `provider_review`: Provider's review text (optional)
- `customer_confirmed`: Customer confirmation status - default: False
- `provider_confirmed`: Provider confirmation status - default: False
- `is_verified`: Service completion verification - default: False
- `base_amount`: Base service amount - default: 0
- `tax_amount`: Tax amount - default: 0
- `service_fee`: Platform service fee - default: 0
- `total_amount`: Total booking amount - default: 0
- `created_at`, `updated_at`: Timestamps (auto-populated)

**Business Logic:**
- Manages booking lifecycle and status transitions
- Handles dual confirmation (customer and provider)
- Manages financial details of bookings
- Supports both date and datetime fields for flexibility

### Availability Model
Provider availability by day of week with time ranges.

**Fields:**
- `provider`: Foreign key to Django User (provider)
- `day_of_week`: Integer field (0=Monday, 6=Sunday) with choices
- `start_time`: Availability start time (TimeField)
- `end_time`: Availability end time (TimeField)
- `is_available`: Availability active status - default: True
- `created_at`, `updated_at`: Timestamps (auto-populated)

**Business Logic:**
- Manages weekly availability patterns
- Enforces unique availability per provider per day
- Supports ordering by day of week and start time

### TimeSlot Model
Time slots linked to specific bookings.

**Fields:**
- `booking`: Foreign key to Booking (one-to-many relationship)
- `start_time`: Time slot start time (DateTimeField)
- `end_time`: Time slot end time (DateTimeField)
- `is_booked`: Booking status - default: False
- `created_at`: Timestamp (auto-populated on creation)

**Business Logic:**
- Represents specific time slots within a booking
- Tracks booking status of specific time ranges
- Maintains link to parent booking

### ServiceLocation Model
Specific locations where services can be provided.

**Fields:**
- `service`: Foreign key to Service
- `location_name`: Location name (max 200 characters)
- `address`: Full address (max 500 characters)
- `latitude`, `longitude`: Geographic coordinates (optional)
- `is_active`: Location active status - default: True
- `created_at`: Timestamp (nullable, for backward compatibility)

**Business Logic:**
- Manages multiple service locations per service
- Stores geographic information for location-based services
- Supports ordering by active status

### BookingCancellation Model
Detailed cancellation information for bookings.

**Fields:**
- `booking`: One-to-one relationship with Booking
- `cancelled_by`: Foreign key to Django User who cancelled
- `cancellation_reason`: Cancellation reason (max 500 chars) - default: ""
- `cancellation_policy_applied`: Applied policy details (max 500 chars, optional)
- `refund_amount`: Amount refunded - default: 0
- `cancelled_at`: Timestamp of cancellation (nullable)
- `created_at`, `updated_at`: Timestamps (auto-populated)

**Business Logic:**
- Provides detailed cancellation information
- Tracks refund amounts
- Maintains link to original booking

## Payments Models

### Payment Model
Payment transactions for bookings.

**Fields:**
- `booking`: One-to-one relationship with Booking
- `transaction_id`: Unique transaction identifier (max 100 chars, unique)
- `amount`: Payment amount (DecimalField with max 10 digits, 2 decimal places)
- `currency`: Currency code - default: "BDT" (max 3 chars)
- `status`: Choice field (pending, processing, completed, failed, refunded, partially_refunded, cancelled) - default: "pending"
- `payment_method`: Choice field (sslcommerz, bkash, nagad, rocket, card, bank_transfer, cash) (optional)
- `payment_gateway_response`: JSONField for gateway response data - default: empty dict
- `payment_gateway_transaction_id`: Gateway's transaction ID (max 100 chars, optional)
- `payment_date`: Date/time of payment (auto-populated on creation)
- `sslcommerz_sessionkey`: SSLCommerz session key (max 100 chars, optional)
- `sslcommerz_validation_response`: SSLCommerz validation response (JSONField, default: empty dict)
- `refund_transaction_id`: Refund transaction ID (max 100 chars, optional)
- `refund_amount`: Amount refunded - default: 0
- `refund_reason`: Reason for refund (optional)
- `refund_date`: Date of refund (nullable)
- `initiated_at`: When payment was initiated (nullable)
- `completed_at`: When payment was completed (nullable)
- `created_at`, `updated_at`: Timestamps (auto-populated)

**Business Logic:**
- Manages payment lifecycle for bookings
- Integrates with multiple payment gateways (SSLCommerz, bKash, Nagad, etc.)
- Handles different payment statuses
- Tracks refund information and history

### Refund Model
Refund processing for payments.

**Fields:**
- `payment`: Foreign key to Payment
- `refund_amount`: Refund amount (DecimalField with max 10 digits, 2 decimal places)
- `refund_reason`: Reason for refund (max 500 chars) - default: ""
- `status`: Choice field (requested, approved, processing, completed, rejected) - default: "requested"
- `refund_transaction_id`: Refund transaction ID (max 100 chars, optional)
- `processed_by`: Foreign key to Django User who processed (nullable)
- `refunded_at`: When refund was completed (nullable)
- `notes`: Additional notes (optional)
- `created_at`, `updated_at`: Timestamps (auto-populated)

**Business Logic:**
- Manages refund workflow
- Validates refund amounts against original payment
- Tracks refund status changes and processing
- Links to the original payment

### Transaction Model
General transaction model for tracking all financial movements.

**Fields:**
- `booking`: Foreign key to Booking
- `payment`: Foreign key to Payment (nullable, blank)
- `transaction_type`: Choice field (booking_payment, service_fee, refund, payout, tax)
- `amount`: Transaction amount (DecimalField with max 10 digits, 2 decimal places)
- `currency`: Currency code - default: "BDT"
- `description`: Transaction description (optional)
- `reference_id`: Reference to related entity (max 100 chars, optional)
- `is_completed`: Completion status - default: False
- `created_at`, `updated_at`: Timestamps (auto-populated)

**Business Logic:**
- Tracks all financial transactions for bookings
- Supports different transaction types
- Maintains transaction history and audit trail
- Links to both bookings and payments

## Analytics Models

### CustomerBehavior Model
User behavior tracking for analytics.

**Fields:**
- `user`: Foreign key to ServiceProfile
- `page_url`: Page that was visited
- `page_title`: Page title
- `session_id`: User session identifier
- `referral_source`: How user arrived at page
- `device_type`: Device used (mobile, desktop, tablet)
- `browser`: Browser information
- `os`: Operating system information
- `viewed_at`: When page was viewed
- `created_at`, `updated_at`: Timestamps

**Business Logic:**
- Tracks user navigation
- Collects behavioral data
- Supports analytics and insights

### PageView Model
Detailed page view tracking.

**Fields:**
- `url`: Page URL
- `title`: Page title
- `user`: Foreign key to ServiceProfile (optional, for authenticated users)
- `session_id`: Session identifier
- `viewed_at`: When page was viewed
- `created_at`, `updated_at`: Timestamps

**Business Logic:**
- Detailed page view tracking
- Supports user journey analysis
- Maintains visit history

## Notifications Models

### Notification Model
User notifications system.

**Fields:**
- `recipient`: Foreign key to ServiceProfile (user receiving notification)
- `actor`: Foreign key to ServiceProfile (user causing notification, optional)
- `verb`: Action that caused notification (e.g., "booked", "rated", "confirmed")
- `target_content_type`: Generic relation to target object type
- `target_object_id`: Generic relation to target object ID
- `target`: Generic foreign key to target object
- `description`: Optional description of notification
- `is_read`: Read status
- `read_at`: When notification was read
- `level`: Notification level (info, success, warning, error)
- `created_at`, `updated_at`: Timestamps

**Business Logic:**
- Manages user notifications
- Supports different notification types
- Tracks read/unread status
- Links to related objects

## Common Model Features

### BaseModel Inheritance
All models inherit from BaseModel which provides:
- `created_at`: Auto-populated on creation
- `updated_at`: Auto-populated on updates
- `is_active`: Soft delete capability (where applicable)

### Validation
- All models implement appropriate field validation
- Business logic validation implemented where required
- Database constraints used for critical validation
- Unique constraints where appropriate

### Indexing
- Proper indexing for frequently queried fields
- Foreign key fields indexed where appropriate
- Composite indexes for multi-field queries
- Performance considerations for large datasets

## Related Documentation

- [See: data/common-model-patterns.md]
- [See: data/relationships.md]
- [See: architecture/component-diagram.md]
- [See: references/glossary.md#data-models]