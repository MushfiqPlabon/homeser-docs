# Known Security Vulnerabilities

Last Updated: 2025-12-06 04:54 AM

This document details current security vulnerabilities identified in the HomeSer service marketplace.

## Critical Vulnerabilities

### 1. Race Condition in Booking Creation
- **Status**: **FIXED - 2025-12-06**
- **Severity**: Critical (CVSS 8.1)
- **Description**: Concurrent booking requests can bypass availability checks and create overlapping bookings
- **Location**: `bookings/views.py` - `BookingViewSet.perform_create()`
- **Impact**: Double-booking scenarios, customer dissatisfaction, provider conflicts
- **Fix Applied**: Added `select_for_update()` on conflicting bookings with proper time range locking

### 2. Missing Transaction Rollback in Payment Webhook
- **Status**: **FIXED - 2025-12-06**
- **Severity**: Critical (CVSS 7.8)
- **Description**: Payment webhook processing lacks atomic transaction wrapper
- **Location**: `payments/views.py` - `webhook()` method
- **Impact**: Inconsistent payment states, potential financial discrepancies
- **Fix Applied**: Added `@transaction.atomic` decorator to webhook method

### 3. Token Storage Key Mismatch
- **Status**: **FIXED - 2025-12-06**
- **Severity**: Critical (CVSS 7.5)
- **Description**: Frontend stores token as "token" but API client retrieves "access_token"
- **Location**: `authStore.jsx` vs `utils/api.js`
- **Impact**: Authentication failures, broken API calls after login
- **Fix Applied**: Standardized all localStorage calls to use "access_token" key

## High Severity Vulnerabilities

### 4. Supabase Sync Failures in Django Signals
- **Status**: **IDENTIFIED - NOT FIXED**
- **Severity**: High (CVSS 7.2)
- **Description**: Signal handlers call Supabase sync without exception handling
- **Location**: `bookings/signals.py`
- **Impact**: Booking creation failures if Supabase unavailable
- **Recommended Fix**: Wrap sync calls in try-except blocks within signals

### 5. Missing Database Locking in Availability Checker
- **Status**: **IDENTIFIED - NOT FIXED**
- **Severity**: High (CVSS 7.1)
- **Description**: Conflict checking lacks database locking between check and create
- **Location**: `bookings/availability_checker.py` - `check_booking_conflicts()`
- **Impact**: Race condition leading to double bookings
- **Recommended Fix**: Use `select_for_update()` when checking conflicts

### 6. Missing Webhook Replay Protection
- **Status**: **IDENTIFIED - NOT FIXED**
- **Severity**: High (CVSS 7.0)
- **Description**: No deduplication for duplicate webhook deliveries from SSLCommerz
- **Location**: `payments/views.py` - `webhook()`
- **Impact**: Duplicate payment processing, incorrect status updates
- **Recommended Fix**: Store and check processed webhook transaction IDs

### 7. Payment Amount Validation Gap
- **Status**: **IDENTIFIED - NOT FIXED**
- **Severity**: High (CVSS 6.8)
- **Description**: Payment creation doesn't verify amount matches booking total
- **Location**: `payments/views.py` - `perform_create()`
- **Impact**: Financial discrepancies, incorrect billing
- **Recommended Fix**: Add validation comparing payment amount to booking.total_amount

## Medium Severity Vulnerabilities

### 8. Frontend Auth Token Refresh Race Condition
- **Status**: **IDENTIFIED - NOT FIXED**
- **Severity**: Medium (CVSS 5.5)
- **Description**: Multiple concurrent 401 responses trigger simultaneous token refresh attempts
- **Location**: `utils/api.js` - response interceptor
- **Impact**: Unnecessary API calls, potential token invalidation
- **Recommended Fix**: Implement refresh request deduplication/queuing

### 9. Booking Duration Validation Missing
- **Status**: **IDENTIFIED - NOT FIXED**
- **Severity**: Medium (CVSS 5.3)
- **Description**: No validation that duration field matches end_time - start_time
- **Location**: `bookings/models.py` - `Booking` model
- **Impact**: Inconsistent data, incorrect billing, scheduling conflicts
- **Recommended Fix**: Add model clean() method to validate duration consistency

### 10. Token Storage in localStorage
- **Status**: **DOCUMENTED - ACCEPTED RISK**
- **Severity**: Medium (CVSS 5.3)
- **Description**: JWT tokens stored in localStorage vulnerable to XSS attacks
- **Location**: Frontend token storage mechanism
- **Impact**: Token theft through XSS
- **Mitigation**: Consider HTTP-only cookies for production

## Remediation Priority

### Completed (2025-12-06)
✅ 1. Fixed booking creation race condition (#1)
✅ 2. Fixed token storage key mismatch (#3)
✅ 3. Added transaction atomic to payment webhook (#2)

### Immediate (Before Production)
4. Add payment amount validation (#7)
5. Implement webhook replay protection (#6)

### High Priority (Current Sprint)
6. Add error handling to Supabase sync (#4)
7. Add locking to availability checker (#5)
8. Fix auth token refresh race condition (#8)

## Related Documentation

- [See: security/common-measures.md]
- [See: security/authentication-system.md]
- [See: deployment/environment-configs.md]
- [See: api/authentication-flow.md]