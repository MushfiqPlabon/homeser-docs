# Glossary

This document defines technical and business terms used throughout the HomeSer service marketplace documentation.

## A

### API (Application Programming Interface)
A set of rules and protocols for building and interacting with software applications. In HomeSer, APIs allow the frontend to communicate with the backend services.

### Authentication
The process of verifying the identity of a user or system. HomeSer uses JWT tokens for API authentication and Supabase for real-time authentication.

### Authorization
The process of determining what resources or actions a user has access to after authentication. HomeSer uses role-based access control (RBAC).

### ADR (Architecture Decision Record)
A document that captures an important architectural decision made during the development of HomeSer, along with the context and consequences of the decision.

## B

### Backend
The server-side component of the HomeSer application, built with Django and Django REST Framework, responsible for processing requests, managing data, and business logic.

### Booking
A reservation made by a customer for a specific service at a specific time. Bookings have statuses like pending, confirmed, in_progress, completed, etc.

### Booking Flow
The complete process from customer searching for services to booking confirmation and completion, including payment processing.

## C

### Caching
The technique of storing frequently accessed data in a temporary storage area (cache) to serve future requests faster. HomeSer uses Redis for server-side caching and React Query for client-side caching.

### CI/CD (Continuous Integration/Continuous Deployment)
Automated processes that allow code changes to be validated and deployed to production. In HomeSer, this involves automated testing and deployment to Vercel.

### Component Architecture
A software architecture pattern where the system is composed of independent, modular components with well-defined interfaces. HomeSer follows a component-based architecture separating concerns into Django apps.

### CORS (Cross-Origin Resource Sharing)
A security feature that allows web pages to make requests to different domains. HomeSer implements CORS to allow communication between the frontend and backend.

## D

### DRY (Don't Repeat Yourself)
A software development principle that aims to reduce repetition of code and promote reusability. Applied to both code and documentation in HomeSer.

### Database Migration
The process of applying changes to the database schema. Django provides migration tools to update the database structure without losing data.

### Dual Authentication System
HomeSer's approach of using both Django JWT authentication for API requests and Supabase authentication for real-time features.

## F

### Frontend
The client-side component of the HomeSer application, built with React, responsible for user interface and user interaction.

### Frontend Pattern
Recurring solutions to common problems in frontend development, such as state management patterns, component design, and API interaction patterns.

## G

### Geospatial Search
A search functionality that takes geographical location into account. HomeSer uses geospatial search to rank providers based on proximity to customers.

### GraphQL
A query language for APIs that allows clients to request exactly the data they need. Note: Not currently used by HomeSer, which uses REST APIs.

## I

### Intelligent Provider Ranking
HomeSer's sophisticated system for ranking service providers using data science libraries (numpy, polars, scikit-learn) that considers factors like location, ratings, and job completion history.

### Integration Testing
Testing the interaction between different components or services. HomeSer includes integration tests to ensure different parts of the system work together correctly.

### Inheritance (Model)
A Django model feature where one model can inherit fields and methods from another model, typically using abstract base classes in HomeSer.

### API Integration
The process of connecting HomeSer's backend with external services like SSLCommerz for payments, Cloudinary for image storage, and Supabase for real-time features.

## L

### Logger
A system component that records events and errors for debugging and monitoring purposes. HomeSer implements structured logging for debugging and auditing.

### Load Testing
Testing that evaluates how the system performs under expected and peak load conditions. Important for ensuring HomeSer's performance under high usage.

## M

### Model
In Django, a Python class that defines the data structure for database tables. HomeSer's models define the structure for users, services, bookings, payments, etc.

### Model Pattern
A recurring approach or design for creating Django models, including common fields, inheritance patterns, and validation approaches used throughout HomeSer.

### Middleware
Software components that sit between the request and response in Django, handling cross-cutting concerns like authentication, compression, and security headers.

## O

### ORM (Object-Relational Mapping)
A technique that allows database queries to be expressed as object-oriented code. Django's ORM is used throughout HomeSer to interact with the PostgreSQL database.

### Optimistic UI
A pattern where the user interface updates immediately based on the expected outcome of an action, with rollback if the action fails. HomeSer uses this for improved user experience.

### O(1) Operation
An operation that takes constant time regardless of input size. HomeSer implements O(1) cached provider ranking for performance optimization.

## P

### Payment Gateway
A service that processes credit card and other payment transactions. HomeSer uses SSLCommerz as its payment gateway.

### Performance Optimization
Techniques and strategies to improve the speed, responsiveness, and efficiency of the HomeSer application.

### Pattern
A recurring solution to a common problem in software design. HomeSer implements various backend, frontend, testing, and data modeling patterns.

### Performance Monitoring
The practice of tracking and analyzing the performance metrics of the HomeSer application to identify and address performance issues.

## Q

### Query Optimization
Techniques to improve the efficiency of database queries, including proper indexing, select_related/prefetch_related usage, and query planning.

## R

### Real-time Updates
Features that provide immediate updates to users without requiring page refreshes. HomeSer implements real-time updates using Supabase PostgreSQL change subscriptions.

### React Query
A library used in HomeSer's frontend for server state management, caching, and optimistic updates.

### RBAC (Role-Based Access Control)
A method of restricting system access based on user roles. HomeSer implements RBAC with customer, provider, and admin roles.

### REST API
An architectural style for building networked applications using HTTP methods. HomeSer's backend provides a REST API for frontend communication.

### RLS (Row Level Security)
Database security feature that restricts access to rows in a table based on user identity or other characteristics. Supabase implements RLS to control data access.

## S

### Service Marketplace
A digital platform that connects service providers with customers who need those services. HomeSer is a service marketplace for home-related services.

### Session Management
The process of maintaining user state between requests. HomeSer uses JWT tokens for session management.

### Soft Delete
A technique where records are marked as inactive rather than being permanently deleted from the database. Implemented in HomeSer for important data preservation.

### SSLCommerz
The payment gateway service integrated into HomeSer for processing payments and transactions.

### Supabase
An open-source Firebase alternative used by HomeSer for PostgreSQL database hosting, authentication, and real-time features.

### State Management
The process of managing and updating application state in frontend development. HomeSer uses Zustand for client state and React Query for server state.

### Supabase Authentication
The authentication system provided by Supabase that integrates with HomeSer for real-time features and user management.

## T

### Token Rotation
The security practice of frequently changing authentication tokens. HomeSer implements JWT token rotation for improved security.

### TTL (Time To Live)
The period of time for which a cached item remains valid. HomeSer uses various TTLs for different types of cached data.

### Test-Driven Development (TDD)
A software development approach where tests are written before the actual code. HomeSer implements testing at multiple levels.

### Third-party Integration
The process of connecting HomeSer with external services like payment gateways, image storage, and real-time systems.

## V

### Vercel
The cloud platform used to deploy and host HomeSer's frontend and backend applications.

### Vite
A fast build tool used in HomeSer's frontend development for development and production builds.

## W

### Webhook
An HTTP callback that allows external services to send real-time information to HomeSer. SSLCommerz uses webhooks to communicate payment status updates.

### WebSocket
A communication protocol that provides full-duplex communication channels over a single TCP connection. Used by HomeSer for real-time updates through Supabase.

## Z

### Zustand
A small, fast, and scalable state management solution used in HomeSer's frontend for client state management.

## Related Documentation

- [See: references/acronyms.md]
- [See: architecture/glossary.md]
- [See: patterns/backend-patterns.md]
- [See: patterns/frontend-patterns.md]