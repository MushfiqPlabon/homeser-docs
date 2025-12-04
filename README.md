# HomeSer Documentation

Welcome to the comprehensive documentation for the HomeSer service marketplace. This documentation follows DRY (Don't Repeat Yourself) principles to ensure each piece of information exists in a single authoritative source, with cross-references as needed.

## Navigation

This documentation is organized by concerns to ensure maintainability and clarity:

### [contributing/](contributing/)
- [Documentation Guidelines](contributing/documentation-guidelines.md) - How to write DRY documentation
- [Template Standard](contributing/template-standard.md) - Standard templates to prevent redundancy
- [Update Procedures](contributing/update-procedures.md) - How to maintain and update docs

### [architecture/](architecture/)
- [System Context](architecture/system-context.md) - High-level system overview
- [Container Diagram](architecture/container-diagram.md) - Major system components
- [Component Diagram](architecture/component-diagram.md) - Internal component relationships
- [Architecture Decisions](architecture/architecture-decisions/) - Decision records with context

### [api/](api/)
- [Common Patterns](api/common-patterns.md) - Reusable API patterns and standards
- [Booking Flow](api/booking-flow.md) - Booking system API flow
- [Payment Flow](api/payment-flow.md) - Payment processing API flow
- [Real-time Events](api/real-time-events.md) - Real-time update patterns
- [Authentication Flow](api/authentication-flow.md) - Authentication API flow

### [security/](security/)
- [Common Measures](security/common-measures.md) - Reusable security principles
- [Authentication System](security/authentication-system.md) - Dual authentication implementation
- [Vulnerabilities](security/vulnerabilities.md) - Known security vulnerabilities
- [Security Measures](security/security-measures.md) - Implemented security measures

### [data/](data/)
- [Common Model Patterns](data/common-model-patterns.md) - Reusable data patterns
- [Models](data/models.md) - Core data models and relationships
- [Relationships](data/relationships.md) - Entity relationships and constraints
- [Constraints](data/constraints.md) - Database constraints and validation

### [deployment/](deployment/)
- [Common Configuration](deployment/common-configuration.md) - Reusable deployment patterns
- [Vercel Setup](deployment/vercel-setup.md) - Vercel deployment configuration
- [Environment Configs](deployment/environment-configs.md) - Environment management

### [performance/](performance/)
- [Common Strategies](performance/common-strategies.md) - Reusable performance patterns
- [Caching Strategy](performance/caching-strategy.md) - Caching implementation and patterns
- [Monitoring](performance/monitoring.md) - Performance monitoring approaches

### [patterns/](patterns/)
- [Backend Patterns](patterns/backend-patterns.md) - Backend implementation patterns
- [Frontend Patterns](patterns/frontend-patterns.md) - Frontend implementation patterns
- [Testing Patterns](patterns/testing-patterns.md) - Testing implementation patterns

### [ai-context/](ai-context/)
- [AI Context Overview](ai-context/README.md) - Bridge for AIs with 2023 knowledge cutoffs
- [Tech Landscape 2025](ai-context/tech-landscape-2025.md) - Current technology state
- [Version Comparison](ai-context/version-comparison-bridge.md) - Then vs now comparison
- [New Patterns](ai-context/new-patterns-2025.md) - Patterns since 2023
- [Deprecated Guidance](ai-context/deprecated-guidance-2023.md) - Outdated patterns to avoid
- [HomeSer Context](ai-context/home-specific-context.md) - Project-specific 2025 context

### [references/](references/)
- [Glossary](references/glossary.md) - Technical term definitions
- [Acronyms](references/acronyms.md) - Acronym definitions
- [External Links](references/external-links.md) - Helpful external references

## Documentation Philosophy

This documentation follows DRY (Don't Repeat Yourself) principles:

1. **Single Source of Truth**: Each piece of information exists in exactly one place
2. **Link Instead of Duplicate**: Cross-reference between documents rather than repeating content
3. **Modular Components**: Reusable documentation modules linked from multiple places
4. **Hierarchical Organization**: Information organized so higher-level docs reference lower-level details

## For AI Agents

When working with the HomeSer codebase:
- Refer to [AGENTS.md](../AGENTS.md) for operational guidelines
- Use this documentation for detailed technical information
- Update relevant documentation when making code changes
- Follow DRY principles when adding new documentation