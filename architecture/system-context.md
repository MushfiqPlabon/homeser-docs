# System Context

This document describes the system context for the HomeSer service marketplace, outlining the high-level system and its relationships with external entities.

## Overview

HomeSer is a service marketplace that connects service providers with customers who can book their services directly online. The system enables customers to browse services, book time slots, make payments, and review service providers.

## External Entities

### Users
- **Customers**: End users who browse and book services
- **Service Providers**: Individuals or businesses offering services
- **Administrators**: System administrators managing the platform

### External Services
- **Supabase**: Database (PostgreSQL) and authentication provider, real-time updates
- **Cloudinary**: Media storage and management
- **SSLCommerz**: Payment gateway for processing transactions
- **Vercel**: Frontend hosting and backend deployment

## System Boundaries

### HomeSer Backend (Django/DRF)
- User authentication and authorization
- Service catalog management
- Booking system and availability management
- Payment processing integration
- Real-time update distribution
- Analytics and reporting

### HomeSer Frontend (React)
- User interface for browsing services
- Booking calendar and scheduling
- Payment processing interface
- Real-time updates display
- User profile management

## Data Flows

### Customer Flow
1. Customer accesses frontend application
2. Browses services through service catalog
3. Selects service and checks provider availability
4. Books time slot through booking system
5. Makes payment through SSLCommerz integration
6. Receives service and provides review

### Provider Flow
1. Provider manages service availability
2. Sets service pricing and terms
3. Accepts/rejects booking requests
4. Receives payment notifications
5. Manages profile and service offerings

## Technical Dependencies

### Database Layer (Supabase PostgreSQL)
- Stores user profiles, services, bookings, payments
- Implements Row Level Security (RLS) for data access control
- Provides real-time change notifications

### Authentication Layer
- JWT-based authentication with access/refresh tokens
- Role-based access control (customer/provider/admin)
- Dual authentication system (Supabase + Django backend)

### Caching Layer (Redis)
- O(1) cached provider ranking calculations
- API response caching with configurable TTL
- Session management

### File Storage (Cloudinary)
- Service image storage and management
- Optimization and delivery of media assets

## Integration Points

### Real-time Updates
- Supabase PostgreSQL change subscriptions for live updates
- Django model synchronization to Supabase via Django signals
- Real-time booking status updates (synced via post_save/post_delete signals)
- Real-time availability updates (synced via post_save signals)
- In-app and email notification system

### Payment Processing
- SSLCommerz integration for transaction processing
- Payment status verification and management
- Refund processing system

### Security Measures
- Multiple security headers
- Rate limiting for API endpoints
- Input validation and sanitization
- Authentication token rotation
- Row Level Security (RLS) in Supabase for data access control
- User-specific data access policies for customers, providers, and admins

## Constraints and Limitations

### Free Tier Constraints
- Supabase: 500MB database, 10K MAUs
- Upstash Redis: 256MB, 500K commands/mo
- Cloudinary: 25 credits/mo, 10MB/file limit
- Vercel: 100 build hrs/mo, 1M invocations/mo

### Performance Targets
- Sub-50ms perceived latency through caching and optimistic updates
- Client-first state management with backend validation
- O(1) operations through intelligent caching strategies

## Related Documentation

- [See: architecture/container-diagram.md]
- [See: security/authentication-system.md]
- [See: deployment/vercel-setup.md]
- [See: references/glossary.md#service-marketplace]