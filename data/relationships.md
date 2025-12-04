# Data Model Relationships

This document describes the relationships between the core data models in the HomeSer service marketplace, including foreign key relationships, cascading rules, and business implications of each relationship.

## Overview

The HomeSer data model relationships form the foundation of the service marketplace, connecting users, services, bookings, payments, and other entities. This document details the relationships between models, their implementation, and business logic implications.

## User and Profile Relationships

### User to ServiceProfile (One-to-One)
```
User ---1:1---> ServiceProfile
```

**Relationship Details:**
- **Type**: One-to-One relationship
- **Field**: `ServiceProfile.user` (primary key)
- **Cascading**: CASCADE (profile deleted when user deleted)
- **On Delete**: CASCADE

**Business Implications:**
- Each user has exactly one service profile
- Profile contains marketplace-specific information
- User authentication is separated from marketplace functionality
- Profile holds user type (customer, provider, admin)

### ServiceProfile to User Type
**Business Rules:**
- ServiceProfile determines user role in marketplace
- Role affects available functionality and permissions
- Provider profiles require additional verification steps
- Admin profiles have system-wide access

## Service Catalog Relationships

### Service to ServiceCategory (Many-to-One)
```
Service ---M:1---> ServiceCategory
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key)
- **Field**: `Service.category` (Foreign Key to ServiceCategory)
- **On Delete**: SET_NULL (category can be deleted without affecting services)
- **Null Allowed**: No (services must have a category)

**Business Implications:**
- Services must belong to a category
- Categories can be hierarchical (parent categories)
- Services appear in search results based on category
- Category changes may affect service visibility

### Service to ServiceProfile (Provider) (Many-to-One)
```
Service ---M:1---> ServiceProfile (provider)
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key)
- **Field**: `Service.provider` (Foreign Key to ServiceProfile)
- **On Delete**: CASCADE (services deleted when provider is deleted)
- **Related Name**: `services_provided`

**Business Implications:**
- Each service belongs to one provider
- Providers can have multiple services
- Service availability tied to provider status
- Provider verification affects all services

### Service to ServiceImage (One-to-Many)
```
Service ---1:M---> ServiceImage
```

**Relationship Details:**
- **Type**: One-to-Many (Foreign Key on child)
- **Field**: `ServiceImage.service` (Foreign Key to Service)
- **On Delete**: CASCADE (images deleted when service is deleted)
- **Related Name**: `images`

**Business Implications:**
- Services can have multiple images
- One primary image per service
- Images hosted on Cloudinary
- Image order affects presentation

### Service to ServiceReview (One-to-Many)
```
Service ---1:M---> ServiceReview
```

**Relationship Details:**
- **Type**: One-to-Many (Foreign Key on child)
- **Field**: `ServiceReview.service` (Foreign Key to Service)
- **On Delete**: CASCADE (reviews deleted when service is deleted)
- **Related Name**: `reviews`

**Business Implications:**
- Services can have multiple reviews
- Reviews contribute to service rating
- Deleted services lose all reviews
- Review validation tied to booking completion

### Service to ServiceLocation (One-to-Many)
```
Service ---1:M---> ServiceLocation
```

**Relationship Details:**
- **Type**: One-to-Many (Foreign Key on child)
- **Field**: `ServiceLocation.service` (Foreign Key to Service)
- **On Delete**: CASCADE (service locations deleted when service is deleted)
- **Related Name**: `service_locations`

**Business Implications:**
- Services can be offered at multiple locations
- Each location has its own geographic coordinates
- Location-based service availability
- Provider can define service areas

## Booking System Relationships

### Booking to ServiceProfile (Customer) (Many-to-One)
```
Booking ---M:1---> ServiceProfile (customer)
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key)
- **Field**: `Booking.customer` (Foreign Key to ServiceProfile)
- **On Delete**: CASCADE (bookings deleted when customer is deleted)
- **Related Name**: `bookings_made`

**Business Implications:**
- Each booking has one customer
- Customers can have multiple bookings
- Customer deletion affects booking history
- Booking status affects customer experience

### Booking to ServiceProfile (Provider) (Many-to-One)
```
Booking ---M:1---> ServiceProfile (provider)
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key)
- **Field**: `Booking.provider` (Foreign Key to ServiceProfile)
- **On Delete**: CASCADE (bookings deleted when provider is deleted)
- **Related Name**: `bookings_received`

**Business Implications:**
- Each booking has one provider
- Providers can have multiple bookings
- Provider deletion affects booking history
- Provider availability affects booking scheduling

### Booking to Service (Many-to-One)
```
Booking ---M:1---> Service
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key)
- **Field**: `Booking.service` (Foreign Key to Service)
- **On Delete**: CASCADE (bookings deleted when service is deleted)
- **Related Name**: `bookings`

**Business Implications:**
- Bookings tied to specific services
- Service deletion affects related bookings
- Service pricing affects booking costs
- Service availability affects booking scheduling

### Booking to TimeSlot (One-to-Many)
```
Booking ---1:M---> TimeSlot
```

**Relationship Details:**
- **Type**: One-to-Many (Foreign Key on child)
- **Field**: `TimeSlot.booking` (Foreign Key to Booking)
- **On Delete**: CASCADE (time slots deleted when booking is deleted)
- **Related Name**: `time_slots`

**Business Implications:**
- Multiple time slots can be associated with one booking
- Time slot availability tracked per booking
- Booking cancellation removes associated time slots
- Time slot constraints affect booking creation

## Payment and Transaction Relationships

### Payment to Booking (One-to-One)
```
Payment ---1:1---> Booking
```

**Relationship Details:**
- **Type**: One-to-One (Foreign Key)
- **Field**: `Payment.booking` (Foreign Key to Booking)
- **On Delete**: CASCADE (payment deleted when booking is deleted)
- **Related Name**: `payment`

**Business Implications:**
- Each booking has one payment
- Payment status affects booking status
- Payment failure may lead to booking cancellation
- Payment refund affects booking history

### Payment to Refund (One-to-Many)
```
Payment ---1:M---> Refund
```

**Relationship Details:**
- **Type**: One-to-Many (Foreign Key on child)
- **Field**: `Refund.payment` (Foreign Key to Payment)
- **On Delete**: CASCADE (refunds deleted when payment is deleted)
- **Related Name**: `refunds`

**Business Implications:**
- One payment can have multiple refunds
- Partial refunds possible against one payment
- Refund status affects payment status
- Financial audit trail maintained

### Transaction to Payment (Many-to-One - Optional)
```
Transaction ---M:1---> Payment (optional)
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key, null allowed)
- **Field**: `Transaction.payment` (Foreign Key to Payment, null allowed)
- **On Delete**: SET_NULL (transaction preserved when payment deleted)
- **Related Name**: `transactions`

**Business Implications:**
- Not all transactions need payments
- Multiple transactions per payment possible
- Financial tracking flexibility
- Comprehensive audit trail

## Review and Rating Relationships

### ServiceReview to ServiceProfile (Customer) (Many-to-One)
```
ServiceReview ---M:1---> ServiceProfile (customer)
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key)
- **Field**: `ServiceReview.customer` (Foreign Key to ServiceProfile)
- **On Delete**: CASCADE (reviews deleted when customer is deleted)
- **Related Name**: `reviews_written`

**Business Implications:**
- Each review has one customer author
- Customers can write multiple reviews
- Customer deletion affects review history
- Review verification tied to customer status

### ServiceReview to Booking (Many-to-One)
```
ServiceReview ---M:1---> Booking
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key)
- **Field**: `ServiceReview.booking` (Foreign Key to Booking)
- **On Delete**: CASCADE (reviews deleted when booking is deleted)
- **Related Name**: `reviews`

**Business Implications:**
- Reviews must be tied to completed bookings
- Prevents review creation without actual service
- Review verification through booking completion
- Service quality tied to actual experience

## Availability and Scheduling Relationships

### Availability to ServiceProfile (Many-to-One)
```
Availability ---M:1---> ServiceProfile (provider)
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key)
- **Field**: `Availability.provider` (Foreign Key to ServiceProfile)
- **On Delete**: CASCADE (availability deleted when provider is deleted)
- **Related Name**: `availability`

**Business Implications:**
- Providers can have different availability per day
- Weekly recurring patterns
- Availability affects booking scheduling
- Provider can have different hours per day

### TimeSlot to ServiceProfile (Many-to-One)
```
TimeSlot ---M:1---> ServiceProfile (provider)
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key)
- **Field**: `TimeSlot.provider` (Foreign Key to ServiceProfile)
- **On Delete**: CASCADE (time slots deleted when provider is deleted)
- **Related Name**: `time_slots`

**Business Implications:**
- Time slots belong to specific providers
- Provider availability generates time slots
- Booking creates link between time slot and booking
- Time slot reuse when booking cancelled

## Notification Relationships

### Notification to ServiceProfile (Recipient) (Many-to-One)
```
Notification ---M:1---> ServiceProfile (recipient)
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key)
- **Field**: `Notification.recipient` (Foreign Key to ServiceProfile)
- **On Delete**: CASCADE (notifications deleted when recipient is deleted)
- **Related Name**: `notifications`

**Business Implications:**
- Users receive notifications to their profile
- Notification history tied to user account
- User deletion affects notification history
- Read/unread status per user

### Notification to ServiceProfile (Actor) (Many-to-One - Optional)
```
Notification ---M:1---> ServiceProfile (actor - optional)
```

**Relationship Details:**
- **Type**: Many-to-One (Foreign Key, null allowed)
- **Field**: `Notification.actor` (Foreign Key to ServiceProfile, null allowed)
- **On Delete**: SET_NULL (notification preserved when actor is deleted)
- **Related Name**: `notifications_from`

**Business Implications:**
- Not all notifications have a specific actor
- System notifications don't require actor
- Actor deletion doesn't affect notification
- Provides context for notification origin

### Notification to Generic Target (Generic Foreign Key)
**Relationship Details:**
- **Type**: Generic Foreign Key
- **Fields**: `target_content_type`, `target_object_id`
- **Related Name**: `notifications`

**Business Implications:**
- Notifications can reference any model
- Flexible notification system
- Maintains reference to notification context
- Supports notifications about any system entity

## Cascading Implications

### Delete Cascading Rules
- **User deletion**: Cascades to profiles, bookings, reviews, and associated data
- **Service deletion**: Cascades to reviews, images, and associated bookings
- **Booking deletion**: Cascades to payments but sets time slot to NULL
- **Payment deletion**: Cascades to associated refunds

### Protection Rules
- **Category deletion**: Preserved but sets service category to NULL
- **Time slot deletion**: Preserved but booking reference set to NULL
- **Payment deletion**: Preserved but transaction reference set to NULL

## Performance Considerations

### Indexing Strategy
- Foreign key fields indexed for query performance
- Frequently joined fields optimized with indexes
- Composite indexes for multi-field queries
- Consider query patterns when adding relationships

### Query Optimization
- Use `select_related` for forward foreign keys
- Use `prefetch_related` for reverse foreign keys and many-to-many
- Consider `only()` and `defer()` for large datasets
- Plan for N+1 query prevention in API design

## Related Documentation

- [See: data/models.md]
- [See: data/common-model-patterns.md]
- [See: architecture/component-diagram.md]
- [See: api/booking-flow.md]
- [See: references/glossary.md#relationships]