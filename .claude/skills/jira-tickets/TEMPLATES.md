# JIRA Tickets — Templates & Reference

## Work-Type Suffixes

| Domain | Suffix | Example |
|--------|--------|---------|
| API / backend | `<base>-API` | `WE-API` |
| UI / frontend | `<base>-UI` | `WE-UI` |
| DB / migration | `<base>-DB` | `WE-DB` |
| Ops / launch | `<base>-OPS` | `WE-OPS` |

## Epic Description Template

```
<One paragraph describing the feature scope for this milestone.>

## Resources
- Functional Requirements: docs/projects/<name>/functional-requirements-<milestone>.md
- PRD: docs/projects/<name>/prd.md
- Tech Design: docs/projects/<name>/tech-design.md
```

## Story Description Template

```markdown
## End Goal

<One paragraph describing the concrete deliverable.>

## Implementation Details

<Specific technical steps: file paths, component names, API contracts, patterns to follow.>

// est: ~N days

## Out of Scope

- <Explicit exclusions>
```
