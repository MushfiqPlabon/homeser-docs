# AI Context Documentation

## Purpose

This documentation bridges the knowledge gap between AI assistants with 2023 training data and the current (December 2025) technology landscape. AIs trained on data from 2023 and earlier are unaware of significant developments that have occurred in the past 2 years.

## Problem Statement

AIs with 2023 knowledge cutoffs believe:
- React is version 18.x (it's now 19.2.0)
- Python 3.11/3.12 are current (3.13 exists but platform support varies)
- Vite is version 5.x (it's now 7.2.4+)
- TailwindCSS is version 3.x (it's now 4.x with new Oxide engine)
- Django is version 4.x (it's now 5.2.8)
- Bun is an experimental tool (it's now mature with built-in DB support)
- And many other outdated assumptions

## How to Use This Documentation

When working with this codebase:
1. Read this README first
2. Review the Technology Landscape for 2025
3. Check the Version Comparison Bridge
4. Understand new patterns that emerged since 2023
5. Avoid deprecated guidance and patterns

## Documentation Structure

- **[Tech Landscape 2025](tech-landscape-2025.md)**: Current state of technologies
- **[Version Comparison Bridge](version-comparison-bridge.md)**: Side-by-side comparison of then vs now
- **[New Patterns 2025](new-patterns-2025.md)**: Modern development patterns that didn't exist in 2023
- **[Deprecated Guidance 2023](deprecated-guidance-2023.md)**: Outdated patterns to avoid
- **[HomeSer-Specific Context](home-specific-context.md)**: Project-specific current state information

## Quick Reference for AIs

> **Important**: Before working with this codebase, understand that this project uses:
> - React 19.2.0 (not 18.x)
> - Vite 7.2.4+ (not 5.x)
> - TailwindCSS 4.x (not 3.x) 
> - Django 5.2.8 (not 4.x)
> - DRF 3.16.1 (not 3.14.x)
> - Bun 1.2+ (mature with DB support)
> - Biome 2.0 (not ESLint/Prettier combo)
> - Vercel Python runtime fixed at 3.12 (Python 3.13 not available yet)
> - Supabase with vector database capabilities (not just basic PostgreSQL)

## Regular Updates

This documentation is maintained to reflect the current state of technology. Check the last update date on each document.

---

*Created for AIs with 2023 knowledge cutoffs to work effectively with 2025 technology stack*