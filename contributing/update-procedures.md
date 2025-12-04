# Update Procedures

This document outlines the procedures for maintaining and updating documentation in the HomeSer project, ensuring documentation stays current with code changes while maintaining DRY principles.

## Pre-Update Checklist

Before making any documentation updates, verify:

### Content Verification
- [ ] Code/implementation matches the documentation content
- [ ] Cross-references in the document are still valid
- [ ] Related documents are aware of the changes being made
- [ ] No duplication is being created unnecessarily

### DRY Compliance
- [ ] Information doesn't already exist elsewhere (if it does, update the authoritative source)
- [ ] Cross-references are used instead of duplicating content
- [ ] Changes don't create inconsistency with related documentation

## Code Change Documentation Process

When making code changes, follow these steps to keep documentation synchronized:

### 1. Planning Phase
- Identify which documentation files will be affected by the code changes
- Consult [AGENTS.md](../../AGENTS.md) for documentation update responsibilities
- Check [Related Documentation Map](../../AGENTS.md#documentation-map-for-ai-agents) in AGENTS.md

### 2. Implementation Phase
- Make code changes
- Update directly affected documentation files
- Review related documentation for secondary effects

### 3. Verification Phase
- Verify documentation updates match the new code implementation
- Check that all cross-references remain valid
- Ensure no broken links were created
- Confirm related documentation is still accurate

### 4. Post-Update Validation
- Run documentation link validation (if automated tools available)
- Verify consistency with other documentation files
- Update any affected navigation or cross-reference links

## Documentation Update Categories

### Minor Updates (typos, small clarifications)
- Update only the affected document
- Verify the change doesn't affect related documents
- No additional review required

### Major Updates (flow changes, new features)
- Update the primary document
- Review and potentially update related documents
- Consider if new documentation is needed
- Verify all cross-references remain valid

### Architectural Updates (system-wide changes)
- Update in all affected documentation areas
- Review the entire documentation tree for consistency
- Update navigation and cross-reference links as needed
- Consider creating new Architecture Decision Records (ADRs)

## Cross-Reference Management

### When Updating a Document that is Referenced Elsewhere
1. Identify all documents that link to the document being updated
2. Verify if those documents need updates due to your changes
3. Update links if the document location or structure changes
4. Update content if the meaning/content of the referenced document changes

### When Creating New Documents
1. Add the new document to the [homeser-docs/README.md](../README.md) navigation
2. Update cross-references in related documents to point to the new document
3. Update [AGENTS.md](../../AGENTS.md) documentation map if appropriate
4. Ensure the new document follows DRY principles

## AI Agent-Specific Update Procedures

### After Code Changes
When AI agents complete code changes, they must:

#### 1. Update Directly Affected Documentation
- Update documentation that describes the modified functionality
- Follow the specific update categories above based on change scope
- Use appropriate templates from [template-standard.md](template-standard.md)

#### 2. Update Related Documentation
- Check for secondary effects of code changes
- Update any related documentation that may be affected
- Verify consistency across related documents

#### 3. Update Navigation and Cross-References
- Add new documents to navigation in [README.md](../README.md)
- Update cross-references in related documents
- Update [AGENTS.md](../../AGENTS.md) documentation map if needed

#### 4. Verify Documentation Integrity
- Ensure no broken links were created
- Verify that all cross-references are still valid
- Check that DRY principles are maintained

### Documentation Update Responsibilities by Change Type

#### Security Changes
- Update: security/ documentation files
- Cross-reference: related API and authentication documentation
- Verify: all security-related cross-references

#### API Changes
- Update: api/ documentation files
- Cross-reference: related security, performance, and real-time documentation
- Verify: API consistency across related documents

#### Architecture Changes
- Update: architecture/ documentation files
- Create: new Architecture Decision Records if appropriate
- Update: cross-references in related documentation areas

#### Performance Changes
- Update: performance/ documentation files
- Cross-reference: related architecture, API, and deployment documentation
- Verify: performance considerations in related areas

## Verification Procedures

### Manual Verification Checklist
- [ ] All cross-references in updated document work correctly
- [ ] Related documents still accurately reflect the updated content
- [ ] No broken internal links in documentation
- [ ] Documentation accurately reflects current code implementation
- [ ] DRY principles are maintained (no unnecessary duplication)

### Automated Verification (if available)
- Run link validation tools
- Check for consistency with codebase
- Validate against documentation templates

## Version Control Procedures

### Commit Messages for Documentation Updates
- Use descriptive commit messages that explain what documentation was updated
- Reference the related code changes if applicable
- Use format: `docs: [brief description of what was updated]`

### Documentation Review Process
- Documentation changes should be consistent with code changes
- Verify accuracy against current implementation
- Ensure related documentation is not invalidated by changes

## Common Update Scenarios

### Adding New Features
1. Create new documentation following appropriate templates
2. Add to navigation in README.md
3. Update AGENTS.md documentation map if needed
4. Add cross-references from related documentation

### Modifying Existing Features
1. Update existing documentation to reflect changes
2. Update any examples or diagrams
3. Verify related documentation remains accurate
4. Update cross-references if needed

### Deprecating Features
1. Mark documentation as deprecated with deprecation date
2. Add references to new replacements if applicable
3. Update navigation to indicate deprecation
4. Update AGENTS.md references to deprecated documentation

### Security Vulnerability Fixes
1. Update vulnerability documentation in security/
2. Update related security measures documentation
3. Verify security implications are reflected in other documentation
4. Update security guidelines as needed