# 0001 - Dual Authentication System

Date: 2025-12-03
Status: Accepted

## Context

HomeSer requires a robust authentication system that provides real-time updates for booking and payment status while maintaining security and performance. The challenge was to choose between a single authentication system that might not provide all required features, or implement multiple systems that could offer comprehensive functionality.

## Decision

We decided to implement a dual authentication system using both Supabase authentication and Django's built-in authentication system. This approach provides the real-time capabilities of Supabase while maintaining the granular control and custom business logic capabilities of Django's authentication.

## Alternatives Considered

### Single Authentication with Supabase Only
- **Pros**: 
  - Simplicity in implementation
  - Built-in real-time capabilities
  - Passwordless and social authentication options
- **Cons**: 
  - Less control over complex authorization logic
  - Potential vendor lock-in
  - Complex custom user profile requirements

### Single Authentication with Django Only
- **Pros**: 
  - Full control over authentication logic
  - Integration with Django admin
  - Custom permission system with django-guardian
- **Cons**: 
  - No built-in real-time updates
  - Would need to implement WebSocket infrastructure
  - More complex to achieve real-time functionality

### Single Authentication with Custom Real-time Layer
- **Pros**: 
  - Full control over both systems
  - Could be optimized for specific needs
- **Cons**: 
  - High development overhead
  - Security concerns with custom implementation
  - Maintenance complexity

## Consequences

### Positive Consequences
- Real-time updates for booking/payment status through Supabase
- Granular role-based access control through Django
- Flexibility to use the best features of both systems
- Ability to gracefully degrade if Supabase is unavailable

### Negative Consequences
- Increased complexity in managing dual authentication
- Potential for inconsistent authentication states
- More complex error handling and debugging
- Additional code to manage both systems

## Implementation

The dual authentication system is implemented as follows:

### Frontend Implementation
- `authStore.jsx` manages both authentication systems
- Falls back to Django-only auth if Supabase unavailable
- Synchronizes authentication states between systems
- Handles token management for both systems

### Backend Implementation
- Django REST Framework JWT for primary authentication
- Supabase client for real-time connection management
- Graceful degradation mechanisms when Supabase is unavailable
- Email verification through both systems

### Integration Points
- User sessions synchronized between systems
- Token validation performed by both systems
- Permission checks utilize Django's system primarily
- Real-time updates delivered through Supabase

## Related Documentation

- [See: security/authentication-system.md]
- [See: api/authentication-flow.md]
- [See: patterns/frontend-patterns.md#authentication]

## References

- [Django Authentication System Documentation](https://docs.djangoproject.com/en/stable/topics/auth/)
- [Supabase Authentication Documentation](https://supabase.com/docs/guides/auth)
- [JWT Authentication Best Practices](https://jwt.io/introduction/)