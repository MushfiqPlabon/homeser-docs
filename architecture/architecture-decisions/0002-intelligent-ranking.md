# 0002 - Intelligent Provider Ranking System

Date: 2025-12-03
Status: Accepted

## Context

HomeSer service marketplace needed an intelligent way to rank service providers for customers based on multiple factors including location, ratings, and availability. The goal was to provide customers with the best possible provider matches while ensuring O(1) performance for search results, especially for the free tier constraints of the platform.

## Decision

We decided to implement an intelligent provider ranking system using data science libraries (numpy, polars, scikit-learn) with O(1) caching. The system uses a multi-factor scoring algorithm that combines geospatial distance, provider ratings, and job completion history to rank providers for customer searches.

## Alternatives Considered

### Simple Rating-Based Ranking
- **Pros**: 
  - Simple to implement
  - Easy to understand
  - Minimal dependencies
- **Cons**: 
  - Ignores important factors like distance
  - Doesn't optimize for customer experience
  - Could result in distant providers being ranked highly

### Database-Level Ranking
- **Pros**: 
  - Leverages database optimization
  - Single query solution
  - Consistent with existing data patterns
- **Cons**: 
  - Complex SQL queries with multiple JOINs
  - Performance issues with large datasets
  - Difficult to implement sophisticated algorithms

### Third-Party Recommendation Service
- **Pros**: 
  - Offloaded complexity
  - Professional recommendation algorithms
  - Scalable solution
- **Cons**: 
  - Additional cost (not suitable for free tier)
  - Vendor lock-in
  - Less control over ranking factors

### Custom Algorithm with Basic Libraries
- **Pros**: 
  - More control than third-party solutions
  - Better performance than database-only approaches
  - Customizable ranking factors
- **Cons**: 
  - Development time and complexity
  - Need to handle caching and performance optimization

## Consequences

### Positive Consequences
- O(1) performance for search results through intelligent caching
- Sophisticated ranking that considers multiple factors
- Better customer experience with relevant results
- Flexibility to adjust ranking algorithm
- Performance optimization within free tier constraints

### Negative Consequences
- Additional dependencies (numpy, polars, scikit-learn)
- Increased complexity in the services component
- Need for sophisticated caching strategy
- More complex testing requirements
- Learning curve for data science libraries

## Implementation

The intelligent ranking system is implemented in `services/intelligence.py` with the following approach:

### Algorithm Components
- **Geospatial Calculation**: Uses Haversine formula to calculate distance between customer and providers
- **Multi-Factor Scoring**: Combines ratings (60% weight) and job count (40% weight) 
- **Distance Factor**: Prioritizes closer providers while considering other factors
- **Normalization**: Uses scikit-learn's MinMaxScaler for different metrics

### Performance Optimization
- **Vectorized Calculations**: Uses numpy for efficient distance calculations
- **DataFrame Operations**: Uses polars for efficient data manipulation
- **O(1) Caching**: Implements Redis caching with 30-minute TTL
- **Extreme Case Handling**: Comprehensive error handling for all potential failures

### Ranking Formula
```
final_score = (0.6 * normalized_rating_score) + (0.4 * normalized_job_score)
```
Where scores are normalized between 0 and 1, with distance factor affecting the final ranking.

## Related Documentation

- [See: api/booking-flow.md] (for how the ranking is used in booking process)
- [See: data/models.md] (for the models involved in ranking)
- [See: performance/caching-strategy.md] (for caching implementation)

## References

- [NumPy Documentation](https://numpy.org/doc/)
- [Polars Documentation](https://pola.rs/)
- [Scikit-learn Documentation](https://scikit-learn.org/stable/)
- [Haversine Formula Wikipedia](https://en.wikipedia.org/wiki/Haversine_formula)