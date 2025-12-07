# Known Security Vulnerabilities

**Last Updated**: 2025-12-07 04:49 AM

**Status**: ✅ **0 Open Bugs** - All critical and high-priority issues resolved. System is production-ready.

---

## Open Issues

*No open security vulnerabilities at this time.*

---

## Accepted Risks

### Token Storage in localStorage
- **Status**: **ACCEPTED RISK**
- **Severity**: Medium (CVSS 5.3)
- **Description**: JWT tokens stored in localStorage vulnerable to XSS attacks
- **Location**: Frontend token storage mechanism
- **Impact**: Token theft through XSS
- **Mitigation**: Consider HTTP-only cookies for production deployment
- **Decision Date**: 2025-12-06

---

## Recently Fixed (Last 5)

### 2025-12-07: Query Optimization in Dashboard
- **Severity**: Low
- **Fix**: Removed unnecessary `select_related("booking")` from payment queries
- **Files**: `bookings/services.py`

### 2025-12-07: Dashboard Code Duplication
- **Severity**: Medium
- **Fix**: Extracted shared methods `_get_booking_stats()` and `_calculate_revenue()`
- **Files**: `bookings/services.py`

### 2025-12-07: Booking Duration Validation
- **Severity**: Medium (CVSS 5.3)
- **Fix**: Added clean() method and serializer validation for 15min-24hr range
- **Files**: `bookings/models.py`, `bookings/serializers.py`

### 2025-12-06: Auth Token Refresh Race Condition
- **Severity**: Medium (CVSS 5.5)
- **Fix**: Implemented mutex pattern with subscriber queue
- **Files**: `utils/api.js`

### 2025-12-06: Supabase Sync Failures
- **Severity**: High (CVSS 7.2)
- **Fix**: Added try-except blocks with logging to all signal handlers
- **Files**: `bookings/signals.py`

---

## Documentation

**For complete fix history**: See [CHANGELOG.md](../../CHANGELOG.md)

**Related Security Documentation**:
- [See: security/common-measures.md]
- [See: security/authentication-system.md]
- [See: security/security-measures.md]
- [See: deployment/environment-configs.md]
- [See: api/authentication-flow.md]

---

**Note**: This document only tracks OPEN vulnerabilities. All fixed issues are moved to CHANGELOG.md to maintain focus on current risks.
