---
project: eo-rule-editor-history
created: 2026-04-10
status: scope-finalized
---

# EO Rule Editor — Inline Change History

## Problem Statement

Platform engineers and SRE managers who work inside the EO rule editor have no way to see recent changes to a rule without navigating away to a separate audit log view. This context-switching slows down incident response: when a rule is misbehaving, the engineer must leave the editor, search the audit log, find the relevant entries, and return. The inline history panel brings the most recent change history directly into the rule editor, eliminating the navigation overhead.

## Goals

- Surface the 5 most recent change events for a rule inline within the rule editor, without requiring navigation to the audit log view
- Reduce time-to-context during incident response for engineers working in the rule editor

## Non-Goals

- Full audit log search or filtering (that is the Audit Log Settings view)
- Creating or modifying audit events — this is display-only
- Mobile or API surface — this is a web UI feature only

## User Stories

- US-01 [P1] As a Platform Engineer, I want to see the 5 most recent changes to a rule displayed inline in the rule editor, so that I can quickly identify whether a recent change may be causing unexpected behavior without leaving my current context.
- US-02 [P1] As an SRE Manager, I want each inline history entry to show the actor, timestamp, and change type, so that I can attribute recent changes to individuals at a glance during incident triage.
- US-03 [P2] As a Platform Engineer, I want a "View full history" link in the inline panel that opens the full audit log filtered to the current rule, so that I can drill down without manually searching.

---

## Affected Surfaces

- EO Rule Editor UI: US-01, US-02, US-03 — new inline history panel component within existing rule editor
- Audit log read API (internal): US-01 — rule editor will query the existing internal audit log API to fetch recent events for a specific rule_id; no new backend endpoint needed, endpoint already exists from EA audit log work
- No database changes required — reads from existing audit_events table

## Dependencies & Constraints

- Depends on EA audit log work being complete — inline history reads from audit_events table and internal API introduced in EA
- No new backend endpoints, DB migrations, or schema changes
- UI only: new React component in the rule editor view

---

## Success Metrics

- Inline history panel adoption: 40% of rule editor sessions include a history panel view within 90 days of GA
- Reduction in navigation from rule editor to audit log settings view (proxy: fewer direct navigations to audit log from rule editor)

## Open Questions

- What is the performance budget for loading the inline history panel? (should not delay rule editor render)
- Should the inline panel be collapsed by default or expanded?

---

## Scope Proposal

### EA Scope ✅

- ✅ US-01 — Inline 5-entry history panel (core feature)
- ✅ US-02 — Actor, timestamp, change type per entry (required for the panel to be useful)
- ✅ US-03 — "View full history" deep link [P2] — small enough to include in EA

### GA Scope ⏩

(none — all stories fit in EA given small scope)
