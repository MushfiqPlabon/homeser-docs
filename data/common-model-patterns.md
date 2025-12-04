# Common Model Patterns

This document describes the common patterns used in the HomeSer data models, following Django best practices and ensuring consistency across the application's data layer.

## Overview

The HomeSer data models follow consistent patterns for structure, validation, relationships, and business logic implementation. This document outlines the standard patterns used across all models in the system.

## Model Structure Patterns

### BaseModel Pattern
All HomeSer models should inherit from a common base model that includes standard fields:

```python
from django.db import models

class BaseModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    is_active = models.BooleanField(default=True)
    
    class Meta:
        abstract = True
```

### Common Field Patterns
- **Timestamps**: All models include `created_at` and `updated_at` fields
- **Active Status**: Most models include `is_active` field for soft deleting
- **UUID Primary Keys**: Use UUIDs for models that require distributed systems considerations
- **String Field Constraints**: Use `max_length=255` for most text fields

## Relationship Patterns

### Foreign Key Relationships
- **Null Handling**: Use `null=True, blank=True` for optional relationships
- **Cascade Behavior**: Explicitly define `on_delete` behavior
- **Reverse Relations**: Use descriptive `related_name` attributes
- **Indexing**: Add `db_index=True` for frequently queried foreign keys

### Many-to-Many Relationships
- **Through Models**: Use intermediate models when additional data is needed
- **Symmetric Relations**: Use `symmetrical=False` when appropriate
- **Indexing**: Consider indexing on join table fields when frequently queried

### One-to-One Relationships
- **Profile Extensions**: Use for extending User model functionality
- **Unique Constraints**: Ensure uniqueness with `unique=True`
- **Optional vs Required**: Consider whether the relationship should be optional

## Validation Patterns

### Model-Level Validation
- **Clean Method**: Override `clean()` method for model-level validation
- **Field Validation**: Use custom validators for complex field validation
- **Cross-Field Validation**: Validate relationships between fields in the same model
- **Business Logic**: Include business rule validation in model validation

### Constraints
- **Database Constraints**: Use `CheckConstraint` for database-level validation
- **Unique Constraints**: Use `UniqueConstraint` for complex unique requirements
- **Conditional Constraints**: Implement conditional validation requirements
- **Performance**: Consider performance impact of complex constraints

## Business Logic Patterns

### Model Methods
- **Action Methods**: Methods that perform actions on the instance
- **Calculation Methods**: Methods that calculate derived values
- **Validation Helpers**: Helper methods for validation logic
- **Property Methods**: Use properties for calculated attributes

### Manager Patterns
- **Custom Managers**: Create custom managers for complex queries
- **Query Optimization**: Include select_related/prefetch_related in managers
- **Soft Delete**: Use managers to handle soft delete logic
- **Active Records**: Default managers that exclude inactive records

## Soft Delete Pattern

```python
class SoftDeleteModel(BaseModel):
    is_active = models.BooleanField(default=True)
    
    def delete(self, *args, **kwargs):
        self.is_active = False
        self.save()
    
    class Meta:
        abstract = True
```

## Audit and Tracking Patterns

### Modification Tracking
- **Auto Timestamps**: Use `auto_now_add=True` and `auto_now=True`
- **Updater Tracking**: Optional field to track who last updated
- **Changes Logging**: Log significant changes to model instances
- **Versioning**: Consider versioning approaches for critical data

### User Attribution
- **Created By**: Track who created a record (optional)
- **Modified By**: Track who last modified a record (optional)
- **Permissions**: Consider permission requirements for attribution
- **Privacy**: Ensure user attribution respects privacy requirements

## Indexing Strategies

### Single Field Indexes
- **Foreign Keys**: Index foreign key fields that are frequently queried
- **Search Fields**: Index fields used in search operations
- **Ordering Fields**: Index fields frequently used for ordering
- **Filtering Fields**: Index fields commonly used in WHERE clauses

### Composite Indexes
- **Multi-Field Queries**: Create composite indexes for multi-field queries
- **Order Matters**: Consider field order in composite indexes
- **Query Patterns**: Analyze actual query patterns for optimal indexes
- **Performance Impact**: Balance index performance with write performance

## Database-Specific Patterns

### PostgreSQL-Specific Features
- **JSON Fields**: Use for flexible data storage when appropriate
- **Array Fields**: Use for storing lists of simple values
- **Full Text Search**: Implement PostgreSQL full-text search when needed
- **Geospatial Features**: Use for location-based functionality

### Optimizations
- **Select Related**: Use in queries to reduce database hits
- **Prefetch Related**: Use for reverse foreign key and many-to-many relationships
- **Only Method**: Use to limit fields fetched when appropriate
- **Values Method**: Use for simple data retrieval needs

## Model Inheritance Patterns

### Abstract Base Classes
- **Common Fields**: Group common fields in abstract base classes
- **Common Methods**: Include common methods in abstract bases
- **Meta Options**: Share common meta options across models
- **Validation**: Include shared validation in abstract bases

### Multi-table Inheritance
- **Use Cases**: When extending existing models with new fields
- **Performance**: Consider performance impact of multi-table joins
- **Querying**: Understand how queries work with multi-table inheritance
- **Maintenance**: Consider maintenance complexity

## Serialization Patterns

### API Serialization Considerations
- **Related Fields**: Plan for how related fields will be serialized
- **Performance**: Consider serialization performance for large datasets
- **Security**: Ensure sensitive fields are not serialized by default
- **Flexibility**: Design models to support flexible serialization

### Query Optimization for Serialization
- **N+1 Prevention**: Design models and queries to prevent N+1 issues
- **Field Selection**: Consider which fields are needed for serialization
- **Nested Serialization**: Plan for nested object serialization needs
- **Caching Considerations**: Consider how serialization affects caching

## Related Documentation

- [See: data/models.md]
- [See: data/relationships.md]
- [See: patterns/backend-patterns.md#data-modeling]
- [See: references/glossary.md#model-patterns]

## Implementation Guidelines

- Always use abstract base classes for common functionality
- Implement proper validation at both model and field levels
- Consider performance implications of relationships and indexes
- Follow Django best practices for model design
- Test model methods and validation thoroughly