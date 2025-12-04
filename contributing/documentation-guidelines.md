# Documentation Guidelines

This document outlines the guidelines for creating and maintaining documentation in the HomeSer project, with a focus on following DRY (Don't Repeat Yourself) principles.

## Core DRY Principles for Documentation

### 1. Single Source of Truth
- Each piece of information should exist in exactly one place
- Use cross-references instead of duplicating content
- When information exists in multiple places, consolidate to a single authoritative source

### 2. Link Instead of Duplicate
- Use `[See: path/to/document.md]` to reference other documentation
- Use `[See: references/glossary.md#term]` for technical terms
- Avoid copying and pasting content between documents

### 3. Modular Components
- Create reusable documentation modules
- Use templates for consistent formatting
- Document patterns and components separately for reuse

## Documentation Structure

### File Organization
- Organize by concerns (architecture, security, api, etc.)
- Use descriptive but concise file names
- Maintain consistent folder structure

### Content Guidelines
- Focus each document on a single concern or topic
- Use consistent formatting and structure
- Include cross-references to related documentation
- Keep documents concise but comprehensive

## Cross-Reference System

### Standard Reference Format
```
[See: path/to/document.md]
[See: references/glossary.md#term]
[See: architecture/system-context.md]
```

### When to Use Cross-References
- When a concept is explained in detail elsewhere
- When using technical terms (link to glossary)
- When referring to architectural decisions
- When referencing common patterns

## Writing Style

### Tone and Language
- Use clear, concise language
- Maintain technical accuracy
- Write for both AI agents and human developers
- Use active voice where possible

### Formatting Standards
- Use Markdown for all documentation
- Follow consistent heading hierarchy (#, ##, ###)
- Use bullet points and numbered lists appropriately
- Include code examples in triple backticks with language specification

## Quality Assurance

### Before Creating New Documentation
1. Search existing documentation for similar content
2. Consider whether existing documentation can be extended instead
3. Check if the information fits as part of a larger document
4. Consider creating a reusable component instead of specific content

### Before Updating Documentation
1. Verify the code/implementation still matches the documentation
2. Update all cross-references if document structure changes
3. Check for broken links in other documents
4. Ensure consistency with related documentation

## Verification Process

### Documentation Link Validation
- Regular checks for broken internal links
- Verification that cross-referenced documents exist
- Updates when document locations change

### Content Freshness
- Include last-updated timestamps
- Schedule periodic reviews for accuracy
- Align with codebase version changes

## AI Agent Specific Guidelines

### Documentation Update Responsibilities
When making code changes, AI agents must:
1. Update affected documentation
2. Verify cross-references remain valid
3. Add new documentation if new concepts are introduced
4. Follow the update procedures in [update-procedures.md](update-procedures.md)

### Redundancy Prevention
AI agents should be vigilant about:
- Creating duplicate information
- Copying content between documents
- Failing to use existing cross-references
- Creating similar documentation in multiple places