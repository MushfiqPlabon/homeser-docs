# HomeSer-Specific Context for AI Assistants

This document provides AI assistants with specific context about the HomeSer project that has evolved since 2023. It details the exact technology stack, patterns, and decisions relevant to this service marketplace application as of December 2025.

## HomeSer Technology Stack Overview

### Backend Stack (Django/DRF)
- **Python Version**: 3.13 locally, but deployed on Vercel with Python 3.12 runtime (fixed, cannot be changed)
- **Django Version**: 5.2.8 (latest security release as of Nov 2025)
- **DRF Version**: 3.16.1 (with enhanced features since 2023)
- **Package Manager**: uv for Python dependencies (not pip/poetry)
- **Database**: Supabase PostgreSQL with vector capabilities (pgvector)

### Frontend Stack (React/Vite)
- **React Version**: 19.2.0 (October 2025 release, third release this year)
- **Build Tool**: Vite 7.2.4+ (not Webpack or older Vite)
- **Styling**: TailwindCSS 4.x+ with Oxide engine (not 3.x)
- **State Management**: Zustand for client state + TanStack Query for server state
- **Router**: React Router 7+ with middleware support
- **Package Manager**: Bun 1.2+ (not npm/yarn) with built-in DB client

### Tooling Stack
- **Linting/Formatting**: Biome 2.0 (not ESLint/Prettier combo)
- **Python Tooling**: uv for package management and sync
- **JavaScript Tooling**: Bun for package management and dev server

## HomeSer Architecture Decisions

### 1. Dual Authentication System
- **Context**: Not traditional single-auth approach from 2023
- **Implementation**: Combines Supabase auth for real-time features + Django JWT for API
- **Why**: Provides real-time capabilities AND granular API control
- **AI Note**: Don't suggest replacing with single auth unless you understand real-time needs

### 2. Intelligent Provider Ranking System
- **Context**: Uses numpy, polars, scikit-learn for O(1) cached calculations (2023 pattern: basic SQL ordering)
- **Implementation**: Geospatial calculations with Haversine formula + multi-factor scoring
- **Caching**: Redis with 30-minute TTL for ranking results
- **AI Note**: This is sophisticated ML-based ranking, not simple SQL ORDER BY

### 3. Real-time Architecture
- **Context**: Not traditional polling approach from 2023
- **Implementation**: Supabase PostgreSQL change subscriptions for live updates
- **Use Cases**: Booking status changes, availability updates, payment confirmations
- **AI Note**: Changes here affect real-time user experience

### 4. Service Marketplace Patterns
- **Context**: Time-slot booking, not cart/product patterns (2023 e-commerce approach)
- **Flows**: Browse → Book Slot → Pay → Review (not Browse → Cart → Checkout → Review)
- **Entities**: Service providers with availability calendars (not inventory/stock)
- **AI Note**: Don't apply traditional e-commerce patterns

## HomeSer Deployment Architecture

### 1. Vercel Deployment
- **Backend**: Django deployed as serverless functions (not traditional server)
- **Frontend**: Static hosting with serverless function integration
- **Python Runtime**: Fixed at 3.12 (Python 3.13 not available on Vercel yet)
- **AI Note**: Serverless constraints affect architecture decisions

### 2. Third-Party Services Integration
- **Supabase**: PostgreSQL with RLS, Auth, Real-time, and vector capabilities
- **Cloudinary**: Image/video storage and AI-powered transformations
- **Upstash Redis**: 256MB plan with 500K commands/month for caching
- **SSLCommerz**: Payment processing (with 2025 security enhancements)
- **Vercel**: Frontend hosting and backend serverless functions

### 3. Free Tier Constraints
- **Database**: 500MB Supabase Postgres limit (affects data modeling)
- **Redis**: 256MB Upstash limit (affects caching strategies)
- **Vercel**: 100 build hrs/mo, 1M invocations/mo (affects function design)
- **AI Note**: All architecture decisions consider these limits

## HomeSer-Specific Patterns & Practices

### 1. Caching Strategy
- **Level 1**: React Query (frontend) with 5min static/30s dynamic cache
- **Level 2**: Redis (backend) with 5-30min TTL for API responses
- **Level 3**: Intelligent ranking cache with 30min TTL using ML libraries
- **AI Note**: Caching is essential for performance within free tier limits

### 2. Client-First State Management
- **Philosophy**: "CUSTOMER IS KING" - optimize for perceived latency
- **Implementation**: Zustand (client state) + optimistic updates + eventual consistency
- **Backend Role**: Data validator and authoritative source
- **AI Note**: Changes should maintain sub-50ms perceived latency

### 3. Performance Optimization
- **Goal**: Sub-50ms perceived latency through caching and optimistic UI
- **O(1) Operations**: Cached provider ranking using data science libraries
- **Query Optimization**: Extensive use of select_related/prefetch_related
- **AI Note**: Performance is customer experience priority

### 4. Security Implementation
- **Dual System**: Supabase (real-time) + Django (API) authentication
- **RLS**: Supabase Row Level Security for data access control
- **JWT**: Django REST Framework JWT with refresh token rotation
- **AI Note**: Security spans both systems, changes affect both

## HomeSer Technical Debt & Known Issues

### 1. Current Security Issues (Documented in vulnerability.md)
- Missing import in authStore.jsx (supabase, hasSupabaseCredentials)
- Webhook security gaps in payment system

### 2. Architecture Evolution Needs
- payments/views.py (410L) still contains business logic (should move to services)
- Frontend has only 6 test files for 20+ components (test coverage gap)
- Some rate limiting missing on authentication endpoints

### 3. Performance Considerations
- Booking system has concurrency checking (race condition prevention needed)
- Dashboard queries may have N+1 issues (requires optimization)
- Intelligent ranking system uses ML libraries (resource intensive)

## Working with HomeSer Codebase

### Before Making Changes:
1. **Check AGENTS.md**: Read the documentation map and update responsibilities
2. **Review AI Context**: Understand 2025 patterns vs 2023 assumptions
3. **Consider Performance**: Will this affect sub-50ms perceived latency?
4. **Check Free Tier**: Does this respect resource limitations?
5. **Security Impact**: How does this affect both auth systems?

### For New Features:
1. **Follow Service Marketplace Patterns**: Time-slot booking, not e-commerce
2. **Implement Client-First State**: Use optimistic updates where appropriate
3. **Consider Caching Strategy**: How will this be cached at multiple levels?
4. **Plan for Real-time**: Should this trigger real-time updates?
5. **Free Tier Optimization**: Design for resource constraints

### Code Quality Standards:
1. **Library-First**: Use existing libraries before custom code (RULES.md)
2. **Zero Redundancy**: No duplicate code (DRY principle)
3. **Test Everything**: New code requires tests
4. **Update Documentation**: Follow the documentation update responsibilities
5. **Use Proper Tools**: uv for Python, bun for JavaScript (not pip/npm)

## Key Files and Their Importance

### Critical Backend Files:
- `services/intelligence.py`: ML-based provider ranking (numpy/polars/scikit-learn)
- `authStore.jsx`: Dual auth system implementation (may have missing imports)
- `supabaseClient.js`: Real-time infrastructure (308 lines, complex)
- `payments/views.py`: Needs refactoring, has business logic in wrong layer

### Architecture Decision Records:
- `architecture-decisions/0001-dual-auth-system.md`: Reason for dual system
- `architecture-decisions/0002-intelligent-ranking.md`: ML-based ranking details

This context information should be considered before making any changes to ensure alignment with HomeSer's 2025 architecture and business requirements. The project has evolved significantly since 2023, with AI-assisted development patterns, serverless architecture, and sophisticated real-time features.