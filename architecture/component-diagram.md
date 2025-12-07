# Component Diagram

This document describes the internal components of the HomeSer service marketplace, showing how the major system modules interact with each other.

## Overview

The HomeSer system is built with a modular architecture where each Django app represents a major business domain. This document details the internal components and their relationships within the backend system.

## Backend Components (Django Apps)

### Accounts Component
- **Purpose**: User authentication, profiles, and permissions
- **Models**:
  - User (extends Django's AbstractUser)
  - ServiceProfile (extends user with role-based functionality)
- **Services**:
  - SupabaseAuthService (handles Supabase authentication)
  - EmailVerificationService (handles email verification)
- **API Endpoints**:
  - User registration and login
  - Profile management
  - Role-based access control
- **Dependencies**: Depends on Django authentication system

### Services Component
- **Purpose**: Service catalog management, provider ranking
- **Models**:
  - Service, ServiceCategory, ServiceImage, ServiceReview
- **Services**:
  - `services/intelligence.py` - Intelligent provider ranking using numpy/polars/scikit-learn
  - ServiceManagementService - CRUD operations for services
- **API Endpoints**:
  - Service listings and search
  - Service creation and updates
  - Intelligent search with provider ranking
- **Dependencies**: Depends on accounts (providers), bookings (availability)

### Bookings Component
- **Purpose**: Time-slot management, booking system, availability
- **Models**:
  - Booking, TimeSlot, Availability, ServiceLocation
- **Services**:
  - `bookings/services.py` - Booking validation and management
  - `bookings/availability_checker.py` - Availability calculation
- **Real-time Integration**:
  - `bookings/signals.py` - Django signal handlers for real-time synchronization to Supabase
  - Pre-save tracking of status changes for notifications
  - Post-save synchronization of booking changes to Supabase for real-time updates
  - Post-delete synchronization to remove records from Supabase
- **API Endpoints**:
  - Booking creation and management
  - Availability checking
  - Booking status updates
- **Dependencies**: Depends on services, accounts, payments

### Payments Component
- **Purpose**: Transaction processing, payment management
- **Models**:
  - Payment, Transaction, Refund
- **Services**:
  - SSLCommerzService - Integration with SSLCommerz payment gateway
  - PaymentValidationService - Payment validation and verification
- **API Endpoints**:
  - Payment initiation
  - Payment status verification
  - Refund processing
- **Dependencies**: Depends on bookings, accounts

### Analytics Component
- **Purpose**: User behavior tracking and analysis
- **Models**:
  - CustomerBehavior
- **Services**:
  - AnalyticsTrackingService - Tracks user behavior
- **API Endpoints**:
  - Analytics data retrieval
- **Dependencies**: Depends on accounts, services, bookings

### Notifications Component
- **Purpose**: Notification management and delivery
- **Models**:
  - Notification
- **Services**:
  - NotificationService - Manages notification creation and delivery
- **API Endpoints**:
  - Notification retrieval and management
- **Dependencies**: Depends on accounts, bookings, services

## Frontend Components (React Structure)

### State Management
- **Zustand Stores**:
  - `stores/authStore.jsx` - Authentication state (dual auth system)
  - Other domain-specific stores
- **React Query**:
  - Server state management
  - Caching and optimistic updates

### API Layer
- **Axios Instance**:
  - `utils/api.js` - Configured with interceptors for auth
  - Token refresh mechanisms
- **API Hooks**:
  - Custom hooks for common operations
  - Error handling utilities

### UI Components
- **Reusable Components**:
  - `components/ui/` - General UI elements
  - `components/` - Business-specific components
- **Page Components**:
  - `pages/` - Route-specific components
  - Layout components

## Component Interactions

### Backend Interactions
```
[Accounts] ←→ [Services] ←→ [Bookings] ←→ [Payments]
     ↓           ↓           ↓           ↓
[Permissions] [Categories] [Availability] [Transactions]
```

### Service Layer Pattern
Each component follows a service layer pattern:
- Views handle HTTP concerns
- Services handle business logic
- Serializers handle data transformation
- Models handle data persistence

### Cross-Component Dependencies

#### Accounts → Other Components
- Provides user authentication
- Supplies role-based permissions
- Manages user profiles

#### Services → Bookings
- Provides service availability data
- Supplies service-specific constraints
- Links providers to their services

#### Bookings → Payments
- Triggers payment creation for bookings
- Provides booking status updates
- Requests payment status verification

#### Payments → Bookings
- Updates booking status based on payment
- Processes refunds affecting bookings
- Provides payment status to booking flows

## Data Flow Components

### Authentication Flow
1. User authentication (accounts)
2. Token creation and validation
3. Permission checking across components
4. Session management with dual auth system

### Booking Flow
1. Service selection (services)
2. Availability checking (bookings)
3. Booking creation (bookings)
4. Payment processing (payments)
5. Real-time updates (notifications)

### Payment Flow
1. Payment initiation (payments)
2. SSLCommerz integration (payments)
3. Booking status update (bookings)
4. Notification (notifications)

## Common Patterns Across Components

### API Pattern Consistency
- All components use DRF ViewSets for standard operations
- Consistent serialization with nested serializers
- Standard permission classes
- Common pagination implementation

### Error Handling
- Centralized exception handling via `homeser/exception_handler.py`
- Consistent error response format
- Customer-friendly error messages

### Caching Strategy
- Redis integration across components
- Consistent cache keys and TTLs
- Cache invalidation patterns

### Real-time Updates
- Supabase PostgreSQL change subscriptions
- Consistent subscription patterns across components
- Client-side handling in frontend
- Django-Supabase synchronization via signals in `bookings/signals.py`
- Sync operations: INSERT, UPDATE, DELETE operations synchronized to Supabase in real-time
- Error handling in sync operations to prevent failures from affecting core functionality
- Status change tracking using pre-save signals
- Notification triggers through Django signal system when booking statuses change

## Component-Specific Architecture

### Intelligent Ranking Component (`services/intelligence.py`)
- Uses numpy, polars, scikit-learn for O(1) cached calculations
- Implements geospatial search with Haversine formula
- Multi-factor scoring combining ratings and job count
- Caching with 30-minute TTL

### Availability Checking Component (`bookings/availability_checker.py`)
- Calculates available time slots based on provider availability
- Checks against existing bookings
- Handles complex time slot calculations

### Real-time Integration Component (`lib/supabaseClient.js` and `homeser/realtime_sync.py`)
- Manages WebSocket connections to Supabase
- Handles reconnection logic
- Implements selective real-time updates
- Manages connection status
- Low-level synchronization via `homeser/realtime_sync.py` functions
- Error logging for sync operations in `homeser/realtime_sync.py`

## Security Components

### Authentication Components
- Dual system: Supabase + Django backend
- JWT token management
- Automatic refresh mechanisms
- Session synchronization

### Authorization Components
- Role-based permissions (customer/provider/admin)
- Object-level permissions using django-guardian
- Custom permission classes
- Decorators for access control

## Performance Components

### Caching Components
- Redis integration across all major components
- O(1) cached operations for provider ranking
- API response caching with configurable TTLs
- Session caching

### Optimization Components
- Database query optimization with select_related/prefetch_related
- Pagination for large datasets
- Efficient serialization
- Lazy loading patterns in frontend

## Related Documentation

- [See: architecture/system-context.md]
- [See: architecture/container-diagram.md]
- [See: patterns/backend-patterns.md]
- [See: references/glossary.md#component-architecture]