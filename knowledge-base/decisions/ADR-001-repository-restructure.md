# ADR-001: Repository Restructure for Post-MVP Phase

⚠️ **STATUS: DRAFT – NOT APPROVED – DO NOT IMPLEMENT**

**Status**: Draft  
**Date**: 2025-01-20  
**Decision Makers**: Development Team

## Context

After completing MVP Stage 1, the repository contained mixed documentation types:
- Active development guides alongside historical discussions
- Strategic vision documents mixed with implementation specs
- Risk of AI tools and developers using outdated architecture references
- Need for clear separation between active work and historical context

## Decision

Restructure `/knowledge-base` into five distinct folders:
- `/active` - Current working documentation and guides
- `/decisions` - Architecture Decision Records (ADRs)
- `/architecture` - Technical architecture and design discussions
- `/product` - Product vision and requirements
- `/archive` - Historical documents and completed phase reports

## Consequences

### Positive
- Clear separation of concerns for documentation
- AI tools can prioritize active documentation
- New developers have structured onboarding path
- Historical context preserved without cluttering active work
- ADR process established for future decisions

### Negative
- Requires updating internal references and links
- Team needs to adapt to new structure
- Existing bookmarks may break

## Implementation Notes
- Baseline branch `archive/pre-step2-baseline` created before restructure
- No code or application structure modified
- All files preserved, only relocated
