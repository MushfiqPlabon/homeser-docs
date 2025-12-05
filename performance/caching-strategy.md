# Caching Strategy

This document details the comprehensive caching strategy implemented in the HomeSer service marketplace to achieve optimal performance while staying within free tier constraints.

## Overview

The HomeSer caching strategy implements a multi-layered approach with different caching mechanisms for different types of data and access patterns. The strategy aims to achieve sub-50ms perceived latency through intelligent caching and optimistic UI updates.

## Cache Architecture

### Multi-Level Cache Hierarchy

#### Level 1: Redis Cache (Primary Cache)
- **Technology**: Redis hosted on Upstash
- **Purpose**: High-performance, distributed caching layer
- **Backend**: Django Redis cache backend
- **Data Types**: Sessions, API responses, computed values, query results

#### Level 2: Database Query Cache
- **Purpose**: Reduce database query load
- **Implementation**: Django ORM query caching features
- **Data Types**: Complex query results with select_related/prefetch_related

#### Level 3: Frontend Cache
- **Technology**: React Query (server state), Zustand (client state)
- **Purpose**: Client-side caching and optimistic updates
- **Data Types**: User data, UI state, API responses

#### Level 4: Browser/CDN Cache
- **Technology**: Browser cache, Vercel CDN
- **Purpose**: Static asset caching and delivery optimization
- **Data Types**: Images, JS/CSS, fonts, static content

## Redis Cache Strategy

### Cache Configuration
[See: deployment/common-configuration.md#redis-settings] for complete Redis configuration details.

### Cache Key Strategy
- **Prefix Organization**: `homeser:{category}:{identifier}:{parameters}`
- **Consistent Naming**: Standardized key naming conventions
- **Parameter Inclusion**: Include relevant parameters in cache keys
- **Expiration Management**: Appropriate TTL for different data types

### Cache Time-to-Live (TTL) Strategy

#### Short-term Cache (5 minutes)
- **Use Case**: Dynamic data that changes frequently
- **Examples**:
  - Service reviews and ratings
  - Real-time booking status updates
  - User session data
- **TTL**: 300 seconds (5 minutes)

#### Medium-term Cache (10 minutes)
- **Use Case**: Data that changes occasionally
- **Examples**:
  - Service listings with reviews
  - Category listings
  - Non-critical search results
- **TTL**: 600 seconds (10 minutes)

#### Long-term Cache (15 minutes)
- **Use Case**: Static or slowly changing data
- **Examples**:
  - Service catalog listings (list view)
  - Category hierarchies
  - Common reference data
- **TTL**: 900 seconds (15 minutes)

#### Extra Long-term Cache (30 minutes)
- **Use Case**: Planned for computationally expensive operations
- **Examples**:
  - Intelligent provider ranking calculations (currently hardcoded but could use this)
  - Complex analytical results
  - Resource-intensive query results
- **TTL**: 1800 seconds (30 minutes)
- **Constant**: `CACHE_EXTRA_LONG` in `homeser/constants.py`

*Note: Cache durations are defined in `homeser/constants.py` as CACHE_SHORT (5 min), CACHE_MEDIUM (10 min), CACHE_LONG (15 min), and CACHE_EXTRA_LONG (30 min). Currently the 30-minute cache is defined as a constant but may be directly hardcoded in some implementations.*

## Specific Caching Implementations

### API Response Caching
**Implementation**: Custom cache decorators and DRF cache mechanisms
- **Cache Key**: `api:{endpoint}:{user_id}:{query_params}`
- **TTL**: Configurable based on endpoint and data volatility
- **Invalidation**: Manual invalidation when data changes
- **Authentication Context**: Different cache for authenticated vs anonymous users

### Database Query Caching
**Implementation**: Django's caching framework with select_related/prefetch_related
- **QuerySet Caching**: Cache complex query results
- **Aggregation Caching**: Cache computed values and counts
- **Pagination Caching**: Cache paginated results
- **Filtering Caching**: Cache filtered query results

### Intelligent Provider Ranking Cache
**Special Implementation**: O(1) cached provider ranking using data science libraries
- **Technology**: numpy, polars, scikit-learn for efficient calculations
- **Algorithm**: Geospatial search with Haversine formula and multi-factor scoring
- **TTL**: 1800 seconds (30 minutes) for cached ranking results
- **Caching Strategy**: Cache results of expensive ranking calculations
- **Cache Key Format**: `search:{lat}:{lon}:{service_type}` where coordinates are rounded to 2 decimal places
- **Cache Bypass**: If cache fails, calculation continues without caching
- **Library-First Implementation**: Uses external libraries as per RULES.md compliance

### Session Caching
**Implementation**: Redis-backed session storage
- **TTL**: Configured to match JWT token lifetimes
- **Data**: User session information and preferences
- **Security**: Secure session data storage and management

## Frontend Caching Strategy

### React Query Caching
**Configuration**:
```javascript
// Default cache configuration
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 15 * 60 * 1000, // 15 minutes
      retry: 1,
      refetchOnWindowFocus: false,
    }
  }
});
```

### Cache Categories
- **Static Data (Long-lived)**: Categories, system settings, static content
  - staleTime: 60 minutes
  - cacheTime: 2 hours

- **User Data (Medium-lived)**: User profiles, settings, preferences
  - staleTime: 15 minutes  
  - cacheTime: 30 minutes

- **Dynamic Data (Short-lived)**: Real-time updates, booking status
  - staleTime: 5 minutes
  - cacheTime: 10 minutes

- **Critical Data (Very Short-lived)**: Payment status, availability
  - staleTime: 2 minutes
  - cacheTime: 5 minutes

### Optimistic Updates
- **Immediate Feedback**: UI updates before server confirmation
- **Rollback Capability**: Proper rollback on server errors
- **User Experience**: Sub-50ms perceived latency
- **Data Consistency**: Eventual consistency with server validation

## Cache Invalidation Strategy

### Automatic Invalidation
- **Model Events**: Invalidate cache when models are saved/updated
- **Signal-Based**: Django signals for cache invalidation
- **Related Data**: Invalidate related cache entries
- **Time-based**: Automatic expiration based on TTL

### Manual Invalidation
- **CRUD Operations**: Manual invalidation after create, update, delete
- **Bulk Operations**: Invalidate cache after bulk changes
- **Admin Actions**: Invalidate cache after administrative changes
- **Migration Updates**: Invalidate cache after data migrations

### Selective Invalidation
- **Specific Keys**: Invalidate specific cache entries
- **Pattern Matching**: Invalidate groups of related entries
- **User-Specific**: Invalidate user-specific cache data
- **Permissions-Based**: Invalidate based on user permissions

## Performance Metrics and Monitoring

### Cache Performance Indicators
- **Hit Rate**: Percentage of cache hits vs misses
- **Response Time**: Time saved through caching
- **Database Load**: Reduction in database queries
- **Memory Usage**: Cache memory utilization

### Monitoring Implementation
- **Cache Statistics**: Track hit rates and performance
- **Expiration Monitoring**: Monitor cache expiration patterns
- **Performance Tracking**: Measure cache effectiveness
- **Alerting**: Alert on cache performance degradation

## Free Tier Optimization

### Redis Usage Optimization
- **Memory Efficiency**: Optimize cache keys and values for size
- **TTL Management**: Prevent memory bloat with appropriate TTLs
- **Key Structure**: Efficient key naming to minimize storage overhead
- **Data Compression**: Compress large cache values when beneficial

### Cost Management
- **Cache Size**: Monitor cache usage to stay within free tier
- **Eviction Policies**: Implement appropriate eviction strategies
- **Performance vs Cost**: Balance performance with cost considerations
- **Efficiency Tracking**: Monitor cache efficiency metrics

## Security Considerations

### Cache Security
- **Data Classification**: Only cache appropriate data types
- **User Privacy**: Don't cache sensitive user information
- **Access Control**: Respect user permissions in cached data
- **Token Security**: Secure handling of authentication tokens

### Cache Poisoning Prevention
- **Input Validation**: Validate data before caching
- **Authorization Checks**: Verify access before caching results
- **Cache Key Security**: Prevent cache key manipulation
- **Data Integrity**: Ensure cached data integrity

## Implementation Examples

### API View Caching
```python
from django.core.cache import cache
from django.views.decorators.cache import cache_page
from django.utils.decorators import method_decorator

# Function-based view caching
@cache_page(60 * 15)  # Cache for 15 minutes
def service_list_view(request):
    # Implementation
    pass

# Class-based view caching
@method_decorator(cache_page(60 * 5), name='dispatch')
class ServiceDetailView(DetailView):
    # Implementation
    pass

# Custom cache implementation
def get_cached_provider_ranking(user_location, service_category):
    cache_key = f"provider_ranking:{user_location}:{service_category}"
    cached_result = cache.get(cache_key)
    
    if cached_result is None:
        # Expensive calculation using numpy/polars/scikit-learn
        result = calculate_provider_ranking(user_location, service_category)
        cache.set(cache_key, result, 60 * 30)  # Cache for 30 minutes
        return result
    
    return cached_result
```

### Frontend Caching
```javascript
// React Query cache usage
const { data: services, isLoading } = useQuery({
  queryKey: ['services', categoryFilter, locationFilter],
  queryFn: () => fetchServices(categoryFilter, locationFilter),
  staleTime: 5 * 60 * 1000, // 5 minutes
  cacheTime: 15 * 60 * 1000, // 15 minutes
});

// Optimistic update example
const mutation = useMutation({
  mutationFn: updateBooking,
  onMutate: async (newData) => {
    await queryClient.cancelQueries(['booking', newData.id]);
    const previous = queryClient.getQueryData(['booking', newData.id]);
    queryClient.setQueryData(['booking', newData.id], newData);
    return { previous };
  },
  onError: (err, newData, context) => {
    queryClient.setQueryData(['booking', newData.id], context.previous);
  },
  onSettled: () => {
    queryClient.invalidateQueries(['booking', newData.id]);
  },
});
```

## Related Documentation

- [See: performance/common-strategies.md] - Strategic performance approach and cross-cutting concerns
- [See: security/security-measures.md] - Security considerations for caching
- [See: performance/monitoring.md]
- [See: architecture/component-diagram.md]
- [See: api/common-patterns.md#caching-strategies]
- [See: references/glossary.md#caching]