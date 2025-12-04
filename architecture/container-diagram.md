# Container Diagram

This document describes the major containers (applications, data stores, external systems) that make up the HomeSer service marketplace system.

## Overview

The system consists of two main applications (frontend and backend) that interact with multiple external services and shared data stores. This document outlines the containers and their relationships.

## Main Application Containers

### HomeSer Backend (Django/DRF)
- **Technology**: Django 5.2.8, Django REST Framework 3.16.1, Python 3.13
- **Responsibility**: 
  - API endpoints for all business functionality
  - Authentication and authorization
  - Business logic processing
  - Data validation and integrity
- **Components**:
  - accounts/ - User management, authentication
  - services/ - Service catalog, provider ranking
  - bookings/ - Booking management, availability
  - payments/ - Payment processing, transaction management
  - analytics/ - User behavior tracking
  - notifications/ - Notification system

### HomeSer Frontend (React/Vite)
- **Technology**: React 19.2.0, Vite 7.2.4, TailwindCSS
- **Responsibility**:
  - User interface rendering
  - Client-side state management
  - Real-time updates processing
  - User interaction handling
- **Components**:
  - Pages for different user flows (customer/provider/admin)
  - Reusable UI components
  - State management with Zustand
  - Server state management with React Query

## Data Storage Containers

### Supabase PostgreSQL
- **Technology**: PostgreSQL database hosted on Supabase
- **Responsibility**:
  - Primary data storage
  - Row Level Security (RLS) enforcement
  - Real-time change notifications
- **Key Tables**:
  - User and ServiceProfile tables
  - Service and ServiceCategory tables
  - Booking and TimeSlot tables
  - Payment and Transaction tables
  - Notification tables

### Redis Cache
- **Technology**: Redis hosted on Upstash
- **Responsibility**:
  - O(1) cached provider ranking calculations
  - API response caching
  - Session management
  - Temporary data storage

### Cloudinary Storage
- **Technology**: Cloud-based image/video storage
- **Responsibility**:
  - Service image storage
  - Media asset optimization and delivery

## External Service Containers

### SSLCommerz Payment Gateway
- **Responsibility**:
  - Payment transaction processing
  - Payment status verification
  - Refund processing

### Supabase Authentication
- **Responsibility**:
  - User authentication
  - Token management
  - User session management

## Communication Patterns

### Backend-to-Database
- Django ORM for PostgreSQL interactions
- Direct Redis operations for caching
- Row Level Security enforcement

### Frontend-to-Backend
- REST API calls via Axios
- JWT authentication tokens
- Real-time subscriptions via Supabase

### Backend-to-External-Services
- SSLCommerz API integration for payments
- Cloudinary API for file storage
- Supabase client libraries for real-time features

### Real-time Communication
- Supabase PostgreSQL change subscriptions
- WebSocket connections for live updates
- Client-side subscription management

## Container Interactions

```
[Customer/Provider/Admin] 
        ↓ (HTTP/HTTPS)
[HomeSer Frontend React]
        ↓ (API calls)
[HomeSer Backend Django]
        ↙ ↓ ↘
[Supabase PostgreSQL] [Redis Cache] [Cloudinary]
        ↑
[External Services: SSLCommerz, Supabase Auth]
```

## Technology Stack per Container

### Backend Container
- **Framework**: Django + DRF
- **Language**: Python 3.13
- **Package Manager**: uv/uvx
- **Database**: Supabase PostgreSQL
- **Cache**: Redis (Upstash)
- **File Storage**: Cloudinary

### Frontend Container
- **Framework**: React 19.2.0
- **Build Tool**: Vite 7.2.4
- **Styling**: TailwindCSS
- **State Management**: Zustand + React Query
- **Package Manager**: bun/bunx

## Deployment Containers

### Vercel Deployment
- **Frontend**: Static hosting
- **Backend**: Serverless functions
- **Environment**: Production configuration
- **Scaling**: Auto-scaling based on demand

## Security Boundaries

### Container Boundary Security
- API authentication with JWT tokens
- CORS configuration for frontend-backend communication
- Rate limiting on public endpoints
- Input validation at container boundaries

### Data Protection
- Encrypted data transmission between containers
- Secure credential management
- Access control at database level (RLS)

## Performance Considerations

### Container Performance
- Caching strategies implemented at multiple levels
- Optimized database queries with select_related/prefetch_related
- Client-side caching with React Query
- CDN for static assets via Vercel

### Communication Optimization
- Efficient API endpoint design
- Minimal data transfer through serialization optimization
- Real-time updates to reduce polling

## Related Documentation

- [See: architecture/system-context.md]
- [See: architecture/component-diagram.md]
- [See: deployment/vercel-setup.md]
- [See: security/common-measures.md]