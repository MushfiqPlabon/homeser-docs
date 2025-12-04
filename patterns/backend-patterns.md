# Backend Implementation Patterns

This document describes the common implementation patterns used throughout the HomeSer backend, focusing on the Django/DRF architecture and service-oriented design.

## Overview

The HomeSer backend follows consistent patterns for API development, data management, business logic implementation, and integration with external services. These patterns ensure maintainability, scalability, and consistency across all Django applications.

## Service Layer Pattern

### Purpose
The service layer pattern separates business logic from API endpoints, making code more maintainable and testable.

### Implementation Structure
```
apps/
├── bookings/
│   ├── services.py          # Business logic classes
│   ├── views.py             # API endpoints (thin layer)
│   └── serializers.py       # Data serialization
```

### Example Implementation
```python
# bookings/services.py
class BookingValidationService:
    @staticmethod
    def validate_provider_availability(provider, date, start_time, end_time):
        """Validate provider availability for the given time slot"""
        # Business logic implementation
        pass
    
    @staticmethod
    def validate_booking_modifiable(booking):
        """Validate if booking can be modified in current state"""
        # Business logic implementation
        pass

# bookings/views.py
class BookingViewSet(viewsets.ModelViewSet):
    def perform_update(self, serializer):
        BookingValidationService.validate_booking_modifiable(self.get_object())
        serializer.save()
```

### Benefits
- **Separation of Concerns**: Business logic separated from API logic
- **Reusability**: Service methods can be used across different endpoints
- **Testability**: Business logic can be tested independently
- **Maintainability**: Changes to business logic don't affect API layer

## API Design Patterns

### ViewSet Pattern
**Purpose**: Consistent CRUD operations with automatic URL routing

```python
from rest_framework import viewsets
from rest_framework.decorators import action

class ServiceViewSet(viewsets.ModelViewSet):
    queryset = Service.objects.all()
    serializer_class = ServiceSerializer
    permission_classes = [IsAuthenticated]
    
    @action(detail=True, methods=['post'])
    def custom_action(self, request, pk=None):
        # Custom endpoint logic
        pass
```

### API Response Standardization
**Purpose**: Consistent API responses across all endpoints

```python
# homeser/exception_handler.py
def custom_exception_handler(exc, context):
    response = exception_handler(exc, context)
    
    if response is not None:
        response.data['success'] = False
        response.data['message'] = str(exc)
        
    return response
```

### Filtering and Search
**Purpose**: Consistent filtering and search patterns across APIs

```python
# Using django-filter for consistent filtering
import django_filters

class ServiceFilter(django_filters.FilterSet):
    category = django_filters.NumberFilter()
    location = django_filters.CharFilter(method='filter_by_location')
    
    class Meta:
        model = Service
        fields = ['category', 'provider', 'is_active']
    
    def filter_by_location(self, queryset, name, value):
        # Custom location filtering logic
        pass
```

## Authentication and Authorization Patterns

### Custom Permission Classes
**Purpose**: Consistent authorization across the application

```python
# accounts/permissions.py
class IsOwnerOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        # Read permissions for any request
        if request.method in permissions.SAFE_METHODS:
            return True
        # Write permissions to the owner
        return obj.owner == request.user

class RoleBasedPermission(permissions.BasePermission):
    def has_permission(self, request, view):
        # Role-based permission logic
        pass
```

### Authentication Decorators
**Purpose**: Consistent role-based access control

```python
# accounts/decorators.py
def require_role(*roles):
    def decorator(view_func):
        def wrapper(request, *args, **kwargs):
            if not request.user.is_authenticated:
                raise PermissionDenied("Authentication required")
            if request.user.serviceprofile.user_type not in roles:
                raise PermissionDenied("Insufficient permissions")
            return view_func(request, *args, **kwargs)
        return wrapper
    return decorator
```

## Data Modeling Patterns

### Soft Delete Implementation
**Purpose**: Preserve data while marking records as inactive

```python
# Common base model with soft delete
class BaseModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    is_active = models.BooleanField(default=True)
    
    class Meta:
        abstract = True

class SoftDeleteManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_active=True)

class Service(BaseModel):
    title = models.CharField(max_length=255)
    # Other fields...
    
    objects = SoftDeleteManager()
    all_objects = models.Manager()  # Access to all objects including soft-deleted ones
```

### UUID Primary Keys
**Purpose**: Distributed system compatibility and security

```python
import uuid
from django.db import models

class BaseModel(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    # Other fields...
    
    class Meta:
        abstract = True
```

## Error Handling Patterns

### Centralized Exception Handling
**Purpose**: Consistent error responses and logging

```python
# homeser/exception_handler.py
def custom_exception_handler(exc, context):
    # Convert validation errors to customer-friendly messages
    if isinstance(exc, ValidationError):
        message = "Invalid input data provided"
        return Response({'error': message, 'code': 'VALIDATION_ERROR'}, status=400)
    
    # Handle permission errors
    if isinstance(exc, PermissionDenied):
        return Response({'error': 'Access denied', 'code': 'PERMISSION_DENIED'}, status=403)
    
    # Call REST framework's default exception handler
    response = exception_handler(exc, context)
    return response
```

### Customer-Friendly Error Messages
**Purpose**: Secure error handling that provides useful information without exposing system details

```python
# In serializers or service classes
def validate_booking(self, data):
    try:
        # Validation logic
        pass
    except IntegrityError:
        raise ValidationError("The requested booking is not available. Please select a different time slot.")
    except Exception as e:
        # Log the actual error for debugging
        logger.error(f"Booking validation failed: {str(e)}")
        # Return generic message to user
        raise ValidationError("Booking validation failed. Please try again.")
```

## Serialization Patterns

### Nested Serialization
**Purpose**: Efficient handling of related data

```python
class ServiceSerializer(serializers.ModelSerializer):
    category = ServiceCategorySerializer(read_only=True)
    provider = ServiceProfileSerializer(read_only=True)
    images = ServiceImageSerializer(many=True, read_only=True)
    
    class Meta:
        model = Service
        fields = '__all__'
        read_only_fields = ('rating_average', 'rating_count')

class BookingSerializer(serializers.ModelSerializer):
    service = ServiceSerializer(read_only=True)
    customer = ServiceProfileSerializer(read_only=True)
    provider = ServiceProfileSerializer(read_only=True)
    
    class Meta:
        model = Booking
        fields = '__all__'
```

### Conditional Serialization
**Purpose**: Different serialization for different user types

```python
class BookingSerializer(serializers.ModelSerializer):
    class Meta:
        model = Booking
        fields = '__all__'
    
    def to_representation(self, instance):
        data = super().to_representation(instance)
        
        # Only show sensitive information to authorized users
        user = self.context['request'].user
        if not (user == instance.customer or user == instance.provider or user.is_staff):
            data.pop('customer_notes', None)
        
        return data
```

## Integration Patterns

### Third-Party Service Integration
**Purpose**: Consistent integration with external services

```python
# payments/services.py
class SSLCommerzService:
    def __init__(self):
        self.store_id = settings.SSLCOMMERZ_STORE_ID
        self.store_pass = settings.SSLCOMMERZ_STORE_PASS
        self.is_sandbox = settings.SSLCOMMERZ_IS_SANDBOX
    
    def create_payment_session(self, booking, amount):
        """Create SSLCommerz payment session"""
        # Integration logic
        pass
    
    def verify_payment(self, transaction_id):
        """Verify payment with SSLCommerz"""
        # Verification logic
        pass
```

### Event-Driven Architecture
**Purpose**: Integration with real-time systems

```python
# In model save methods or signal handlers
def save(self, *args, **kwargs):
    is_new = self.pk is None
    old_status = None
    
    if not is_new:
        old_status = type(self).objects.get(pk=self.pk).status
    
    super().save(*args, **kwargs)
    
    # Trigger real-time updates
    if old_status != self.status:
        self.trigger_status_update_webhook()
```

## Query Optimization Patterns

### Efficient Database Queries
**Purpose**: Optimize performance and reduce database load

```python
class BookingViewSet(viewsets.ModelViewSet):
    def get_queryset(self):
        user = self.request.user
        user_type = user.serviceprofile.user_type
        
        queryset = Booking.objects.all()
        
        # Apply role-based filtering efficiently
        if user_type == 'customer':
            queryset = queryset.filter(customer=user.serviceprofile)
        elif user_type == 'provider':
            queryset = queryset.filter(provider=user.serviceprofile)
        # Admins see all bookings
        
        # Optimize queries with select_related and prefetch_related
        queryset = queryset.select_related(
            'customer', 'provider', 'service'
        ).prefetch_related(
            'service__images', 'service__category'
        )
        
        return queryset
```

### Caching Query Results
**Purpose**: Reduce database load for expensive queries

```python
from django.core.cache import cache
from django.conf import settings

def get_ranked_providers(location, service_category):
    cache_key = f"ranked_providers:{location}:{service_category}"
    cached_result = cache.get(cache_key)
    
    if cached_result is None:
        # Expensive computation using data science libraries
        result = compute_provider_ranking(location, service_category)
        
        # Cache for 30 minutes using intelligent ranking system
        cache.set(cache_key, result, settings.PROVIDER_RANKING_CACHE_TIMEOUT)
        return result
    
    return cached_result
```

## Asynchronous Task Patterns

### Celery Task Integration (Future Implementation)
**Purpose**: Handle long-running operations without blocking API responses

```python
# For future use - background task patterns
from celery import shared_task

@shared_task
def send_booking_confirmation_email(booking_id):
    """Send booking confirmation email asynchronously"""
    booking = Booking.objects.get(id=booking_id)
    # Email sending logic
    pass
```

## Testing Patterns

### Service Layer Testing
**Purpose**: Isolate and test business logic

```python
# bookings/tests_services.py
class BookingValidationServiceTest(TestCase):
    def setUp(self):
        self.provider = ServiceProfileFactory(user_type='provider')
        self.customer = ServiceProfileFactory(user_type='customer')
        self.service = ServiceFactory(provider=self.provider)
    
    def test_validate_provider_availability(self):
        # Test business logic directly
        result = BookingValidationService.validate_provider_availability(
            self.provider, 
            date.today(), 
            "10:00:00", 
            "12:00:00"
        )
        self.assertTrue(result)
```

### API Testing
**Purpose**: Test API endpoints with proper authentication

```python
# bookings/tests_views.py
class BookingViewSetTest(APITestCase):
    def setUp(self):
        self.user = UserFactory()
        self.client = APIClient()
        self.client.force_authenticate(user=self.user)
    
    def test_create_booking(self):
        # Test API endpoint with authentication
        response = self.client.post('/api/bookings/', self.valid_payload)
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
```

## Performance Patterns

### Caching Strategies
**Purpose**: Implement multi-level caching for performance

```python
# In views or service classes
from django.core.cache import cache
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)  # Cache for 15 minutes
def service_list_view(request):
    # Implementation
    pass

# Or in class-based views
class CachedServiceListView(ListView):
    @method_decorator(cache_page(60 * 15))
    def dispatch(self, *args, **kwargs):
        return super().dispatch(*args, **kwargs)
```

## Related Documentation

- [See: patterns/frontend-patterns.md]
- [See: patterns/testing-patterns.md]
- [See: architecture/component-diagram.md]
- [See: data/common-model-patterns.md]
- [See: performance/caching-strategy.md]
- [See: references/glossary.md#backend-patterns]