---
project: {{project-name}}
created: {{date}}
status: draft
design-mocks:
---

# {{Project Name}} — PRD

## Problem Statement
<!--
What problem are we solving?
Who has this problem? (persona, scale, frequency)
What happens today without this solution?
-->

## Goals
<!--
3-5 measurable outcomes. Not features — results.
Each goal should be verifiable after launch.
-->
-

## Non-Goals
<!--
What are we explicitly NOT doing in this scope?
Required — if it's not listed here, it may be assumed in scope.
-->
-

## User Stories
<!--
Format: US-XX [P0/P1/P2] As a <persona>, I want <capability> so that <outcome>.
Example: US-01 [P0] As a compliance officer, I want to filter audit logs by date range so that I can pull evidence for a specific audit window.

Priority tags:
  [P0] — must have; feature is useless without this
  [P1] — important; should ship soon but can be deferred
  [P2] — nice to have; cut if needed

Do NOT pre-split into EA/GA — that is determined downstream by /scope-propose.
-->
-

---
<!-- The following sections are filled by engineering via /prd --enrich -->

## Affected Surfaces
<!-- To be populated by: /prd --enrich --project {{project-name}} -->
<!-- Format: - <surface>: <which stories touch it> (<file found in codebase>) -->
-

## Dependencies & Constraints
<!-- To be populated by: /prd --enrich --project {{project-name}} -->
<!-- Format: - <finding> — <reason> (found: <file path>) -->
-

---

## Success Metrics
<!--
How do we know this is working?
Include leading indicators (adoption) and lagging indicators (outcome).
-->
-

## Open Questions
<!--
Unresolved decisions. No need to answer them here — just name them.
-->
-

---
<!-- The following section is appended by /scope-propose after /prd --enrich completes -->

## Scope Proposal
<!-- Run /scope-propose --project {{project-name}} to populate this section -->
