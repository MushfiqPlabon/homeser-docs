# Frontend Implementation Patterns

This document describes the frontend implementation patterns used in the HomeSer service marketplace, focusing on React 19.2.0 with Zustand and React Query as the primary state management solutions.

## Overview

The HomeSer frontend follows modern React 19 patterns with a focus on user experience, performance, and maintainability. The architecture combines client state (Zustand) with server state (React Query) to create a responsive, optimistic UI experience.

## State Management Patterns

### Combined State Management
- **Client State**: Zustand for client-side state management (user data, UI state, preferences)
- **Server State**: React Query for server state management (API data, caching, mutations)
- **Integration**: Zustand handles user session and UI state, React Query manages server data with caching and optimistic updates

### Zustand Store Patterns
- **Global State**: Centralized stores for authentication, user profile, and persistent UI state
- **Slice Pattern**: Organized store structure with related functionality grouped into slices
- **Middleware**: DevTools integration for debugging and development
- **Hydration**: Proper handling of initial state from localStorage or API

```javascript
// Example pattern from authStore.jsx
export const useAuthStore = create(
  devtools((set, get) => ({
    // State properties
    user: null,
    token: localStorage.getItem("token") || null,
    loading: false,
    
    // Actions with proper error handling
    login: async (email, password) => {
      set({ loading: true });
      try {
        // API call
        const response = await api.login({ email, password });
        // Update state
        set({ user: response.user, token: response.token });
      } catch (error) {
        // Error handling
        set({ loading: false });
      }
    }
  }))
);
```

## API Integration Patterns

### React Query Usage
- **Data Fetching**: useQuery for server state with automatic caching and revalidation
- **Mutations**: useMutation for write operations with optimistic updates
- **Caching Strategy**: 5-minute cache for static data, 30-second cache for dynamic data
- **Error Handling**: Centralized error handling with user-friendly messages

### Optimistic Updates Pattern
- **Immediate Feedback**: UI updates immediately with optimistic data
- **Rollback Capability**: Automatic rollback on API failure
- **Cache Invalidation**: Proper cache management after mutations

```javascript
// Example from optimisticUpdates.js
export const createOptimisticMutation = (queryClient, queryKey) => ({
  onMutate: async (newData) => {
    await queryClient.cancelQueries({ queryKey });
    const previous = queryClient.getQueryData(queryKey);
    
    // Optimistically update cache
    queryClient.setQueryData(queryKey, (old) => {
      if (Array.isArray(old)) {
        return [
          ...old,
          { ...newData, id: `temp-${Date.now()}`, _optimistic: true },
        ];
      }
      return old;
    });
    
    return { previous };
  },
  onError: (_err, _newData, context) => {
    if (context?.previous) {
      queryClient.setQueryData(queryKey, context.previous);
    }
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey });
  },
});
```

## Customer Experience Patterns

### User-Friendly Error Handling
- **Technical to User Translation**: Conversion of technical errors to user-friendly messages
- **Context-Aware Messages**: Different messages for different types of errors (network, auth, server)
- **Consistent Experience**: Standardized error presentation across the application

### Loading States
- **Perceived Performance**: Informative loading messages to improve user experience
- **Action-Specific Messages**: Contextual loading messages for different operations
- **Progressive Enhancement**: Loading states that inform without alarming

## Real-time Integration Patterns

### Supabase Integration
- **Dual Authentication System**: Integration with both Supabase and Django authentication
- **Graceful Degradation**: Functionality that works when Supabase is unavailable
- **Connection Monitoring**: Real-time connection status tracking

### Real-time Subscriptions
- **Booking Updates**: Real-time booking status changes for providers and customers
- **Notification System**: Real-time user notifications
- **Channel Management**: Proper channel setup and cleanup to prevent memory leaks

### Event Handling
- **Automatic Invalidation**: React Query cache invalidation on real-time events
- **Connection Recovery**: Automatic reconnection handling
- **Status Monitoring**: Connection status visibility to users

## Component Architecture

### Reusability
- **Component Composition**: Building complex UIs from smaller, reusable components
- **Hook-based Logic**: Custom hooks for shared logic between components
- **Template Standard**: Consistent component structure for maintainability

### Performance Optimization
- **Memoization**: Proper use of React.memo and useMemo where appropriate
- **Code Splitting**: Component lazy loading for improved initial load times
- **Bundle Optimization**: Tree-shaking and smaller bundle sizes

## Security Patterns

### Token Management
- **Secure Storage**: JWT token storage and management
- **Automatic Refresh**: Seamless token refresh with fallback mechanisms
- **Cleanup Procedures**: Proper token cleanup on logout

### Input Validation
- **Client-Side Validation**: Immediate user feedback with backend verification
- **Sanitization**: Proper input sanitization before API calls
- **Error Prevention**: Proactive validation to prevent errors

## Testing Patterns

### Unit Testing
- **Component Testing**: Testing individual components with React Testing Library
- **Hook Testing**: Testing custom hooks independently
- **Store Testing**: Testing Zustand store actions and state changes

### Integration Testing
- **API Integration**: Testing API calls and data flow
- **State Management**: Testing state updates and caching behavior
- **Real-time Features**: Testing real-time subscription logic

## Performance Patterns

### Caching Strategy
- **Query-based Caching**: React Query's caching based on query keys
- **Optimistic Updates**: Immediate UI updates with eventual consistency
- **Background Sync**: Automatic data synchronization

### Lazy Loading
- **Route-based Splitting**: Component splitting at route level
- **Conditional Loading**: Loading components based on conditions
- **Resource Prioritization**: Critical resources loaded first

## Internationalization and Accessibility

### User Experience Focus
- **Customer-First Design**: All patterns prioritize user experience
- **Accessibility Standards**: Compliance with accessibility guidelines
- **Responsive Design**: Mobile-first approach with responsive layouts

## Related Documentation

- [See: architecture/component-diagram.md]
- [See: performance/caching-strategy.md]
- [See: security/authentication-system.md]
- [See: api/real-time-events.md]
- [See: references/glossary.md#frontend-patterns]