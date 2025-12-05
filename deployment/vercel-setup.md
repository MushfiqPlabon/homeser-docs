# Vercel Setup

This document describes the deployment configuration for the HomeSer service marketplace on Vercel, including both frontend hosting and backend serverless functions.

## Overview

HomeSer is deployed on Vercel with the frontend application hosted as a static site and the Django backend deployed as serverless functions. This setup provides scalability, performance, and cost-effectiveness while staying within free tier constraints.

## Frontend Deployment Configuration

### Vercel Project Configuration
The frontend is deployed using the `vercel.json` configuration file in the `homeser-frontend-react` directory:

```json
{
  "version": 2,
  "name": "homeser-frontend",
  "framework": "vite",
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "dist"
      }
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "dist/index.html"
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options", 
          "value": "DENY"
        },
        {
          "key": "X-XSS-Protection",
          "value": "1; mode=block"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        }
      ]
    },
    {
      "source": "/(.*).(png|jpg|jpeg|gif|svg|ico|webp|avif)$",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    },
    {
      "source": "/(.*).(js|css)$",
      "headers": [
        {
          "key": "Cache-Control", 
          "value": "public, max-age=31536000, immutable"
        }
      ]
    }
  ]
}
```

### Build Process
- **Framework**: Vite is used for build process
- **Build Command**: `bun run build` (or `vite build`)
- **Output Directory**: `dist/` directory
- **Environment**: Production environment variables during build

### Static Asset Optimization
- **Images**: Optimized with long-term caching (1 year with immutable)
- **JS/CSS**: Versioned with content hashes and long-term caching
- **Fonts**: Optimized and cached appropriately
- **Icons**: Efficient format selection and caching

### SPA Routing Configuration
- **Catch-all Route**: All routes except static assets redirect to `index.html`
- **Static Asset Preservation**: Static assets served directly from file system
- **404 Handling**: Proper 404 handling for SPA routing

## Backend Deployment Configuration

### ASGI Application Setup
The Django backend is deployed as an ASGI application using Channels and Uvicorn, configured via `vercel.json`:

```json
{
  "builds": [{"src": "homeser/asgi.py", "use": "@vercel/python"}],
  "routes": [{"src": "/(.*)", "dest": "homeser/asgi.py"}]
}
```

### Python Runtime Configuration
- **Runtime Version**: Python 3.13 (matches development environment)
- **ASGI Server**: Uvicorn with Channels for WebSocket support
- **Timeout**: 60 seconds maximum request handling time
- **Package Management**: Uses `uv` for dependency management
- **Deployment Type**: Serverless functions with ASGI support

### API Routing
- **API Gateway**: All `/api/*` routes handled by Django ASGI application
- **Health Checks**: Available at `/health` endpoint for monitoring
- **Static Files**: Collected and served by the ASGI server
- **WebSocket Support**: Ready for real-time features via Channels

## Environment Configuration

### Environment Variables Setup
Both frontend and backend require environment variables configured in Vercel:

#### Frontend Environment Variables
```
VITE_API_BASE_URL=https://homeser-backend-xyz.vercel.app/api/
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
VITE_ENABLE_REALTIME=true
```

#### Backend Environment Variables
```
DJANGO_SETTINGS_MODULE=homeser.settings
SECRET_KEY=your-secret-key
DEBUG=False
FRONTEND_URL=https://your-frontend-domain.vercel.app
ALLOWED_HOSTS=.vercel.app, homeser-backend-xyz.vercel.app
DATABASE_URL=postgresql://user:pass@host:6543/db
```
[See: deployment/common-configuration.md#database-configuration] for DATABASE_URL format details
[See: deployment/environment-configs.md] for complete environment variable reference

### Environment Variable Security
- **Secret Management**: Use Vercel's secure environment variable storage
- **No Git Commits**: Environment variables never committed to repository
- **Separate Environments**: Different values for development/staging/production
- **Validation**: Environment variable validation in application startup

## Performance Optimization

### Frontend Performance
- **Code Splitting**: Automatic code splitting with Vite
- **Bundle Optimization**: Tree-shaking and dead code elimination
- **Asset Optimization**: Image optimization and compression
- **CDN Distribution**: Global CDN for static assets

### Backend Performance
- **Cold Start Minimization**: Optimized dependency loading
- **Connection Pooling**: Efficient database connection management
- **Caching**: Redis integration for performance caching
- **Static Files**: Proper static file collection and serving

## Monitoring and Observability

### Vercel Analytics
- **Performance Metrics**: Page load times, API response times
- **Error Tracking**: Automatic error tracking and reporting
- **Usage Analytics**: Traffic and usage patterns
- **Build Insights**: Build performance and optimization opportunities

### Custom Monitoring
- **Health Endpoints**: `/health` endpoint for uptime monitoring
- **Performance Metrics**: API response time tracking
- **Error Logging**: Structured logging for debugging
- **User Analytics**: Integration with analytics services

## Deployment Workflow

### CI/CD Pipeline Integration
- **Git Integration**: Automatic deployments on git push
- **Branch Deployments**: Preview deployments for feature branches
- **Production Deployments**: Deployments to production on main branch
- **Rollback Capability**: Easy rollback to previous versions

### Build Process
1. **Install Dependencies**: Using bun for frontend, uv for backend
2. **Run Tests**: Automated testing during build
3. **Build Assets**: Frontend build process with optimization
4. **Deploy**: Deployment to Vercel infrastructure
5. **Verification**: Post-deployment smoke tests

### Custom Domains
- **Frontend Domain**: Custom domain for frontend application
- **Backend Domain**: Custom domain for backend API
- **SSL Certificates**: Automatic SSL certificate management
- **DNS Configuration**: Proper DNS setup for optimal performance

## Database Configuration

### Supabase Row Level Security (RLS)
The application includes comprehensive Row Level Security policies stored in `supabase_rls_policies.sql`:

**Apply these policies in Supabase SQL Editor after running Django migrations:**

**Tables with RLS Enabled:**
- `accounts_serviceprofile` - User profile data with role information
- `services_service` - Service listings with provider associations
- `services_servicecategory` - Service categories (public access)
- `bookings_booking` - Booking records with customer/provider associations
- `bookings_availability` - Provider availability schedules
- `payments_payment` - Payment records linked to bookings

**Customer Policies:**
- `customers_read_own_profile`: Can read their own profile via `user_id = auth.uid()`
- `customers_update_own_profile`: Can update their own profile via `user_id = auth.uid()`
- `customers_read_own_bookings`: Can read bookings they made via `customer_id = auth.uid()`
- `customers_create_bookings`: Can create bookings with `customer_id = auth.uid()`
- `customers_update_own_bookings`: Can update their own bookings via `customer_id = auth.uid()`

**Provider Policies:**
- `providers_manage_own_services`: Can manage services they provide via `provider_id = auth.uid()`
- `providers_read_own_bookings`: Can read bookings for their services via `provider_id = auth.uid()`
- `providers_update_own_bookings`: Can update bookings for their services via `provider_id = auth.uid()`
- `providers_manage_availability`: Can manage their own availability via `provider_id = auth.uid()`

**Admin Policies:**
- `admins_full_access_profiles`: Full access to profiles if user is admin
- `admins_full_access_services`: Full access to services if user is admin
- `admins_full_access_bookings`: Full access to bookings if user is admin

**Public Policies:**
- `public_read_active_services`: Anyone can read active/verified services where `is_active = true AND is_verified = true`
- `public_read_categories`: Anyone can read active categories where `is_active = true`

**Payment Policies:**
- `users_read_own_payments`: Users can read payments linked to their bookings (they are customer or provider for the booking)

**Performance Indexes:**
- Indexes specifically created to optimize RLS queries: `idx_serviceprofile_user_type`, `idx_service_active_verified`, `idx_booking_customer`, `idx_booking_provider`

### Resource Usage Optimization
- **Function Size**: Optimized for Vercel's free tier limits
- **Build Minutes**: Under 100 build minutes per month
- **Serverless Invocations**: Under 1M invocations per month
- **Data Transfer**: Optimized for free tier data transfer limits

### Cost Management
- **Resource Monitoring**: Monitor resource usage to stay within limits
- **Efficiency Optimization**: Optimize functions for minimum resource usage
- **Caching Strategy**: Use caching to reduce serverless function invocations
- **Static Asset Strategy**: Serve static assets directly when possible

## Troubleshooting

### Common Deployment Issues
- **Cold Start Issues**: Optimize initial dependency loading
- **Timeout Issues**: Optimize performance within timeout limits
- **Memory Issues**: Monitor memory usage and optimize accordingly
- **Environment Issues**: Verify environment variables are properly set

### Debugging Strategies
- **Build Logs**: Review build logs for errors
- **Runtime Logs**: Check runtime logs for issues
- **Environment Verification**: Confirm environment variables
- **Dependency Issues**: Verify dependencies are properly installed

## Related Documentation

- [See: deployment/common-configuration.md]
- [See: deployment/environment-configs.md]
- [See: performance/monitoring.md]
- [See: architecture/container-diagram.md]