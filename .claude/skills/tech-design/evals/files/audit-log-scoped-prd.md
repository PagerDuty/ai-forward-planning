---
project: eo-audit-log
created: 2026-04-01
status: approved
---

# EO Audit Log — PRD

## Problem Statement

PagerDuty's Event Orchestration currently provides no mechanism to record, query, or export a history of changes made to orchestration rules, routers, or global configurations. Enterprise customers — particularly those in financial services, healthcare, and government — require all operational tooling to meet change management and audit standards. Without an audit log, EO fails procurement and security reviews, preventing deployment at scale. When an EO rule change causes unexpected behavior, there is no way to identify what changed, who changed it, or when, making root cause analysis slow and difficult.

## Goals

- Capture a structured, queryable record of every change to EO configuration including actor, timestamp, change type, and before/after state
- Enable compliance officers to produce audit reports without manual data extraction
- Unblock enterprise sales by satisfying audit and compliance requirements that currently prevent EO deployment in regulated industries
- Reduce mean time to identify root cause for rule-change-related incidents by 40%

## Non-Goals

- Automated rollback from audit history (future consideration)
- Change approval workflows or gating (future consideration)
- Cross-product audit correlation with PagerDuty Incident events (future consideration)
- AI anomaly detection on audit log patterns (future consideration)

## User Stories

- US-01 [P1] As a Compliance Officer, I want a full history of changes made to EO rules, routers, and global orchestrations, so that I can satisfy internal and external audit requirements.
- US-02 [P1] As an SRE Manager, I want to see who made a change to a specific rule and when, so that I can attribute configuration changes to individuals during an incident review.
- US-03 [P1] As a Platform Engineer, I want to view the before and after state of a rule change, so that I can quickly understand what was modified and revert if necessary.
- US-04 [P1] As a Platform Engineer, I want to filter and search the audit log by date range, user, orchestration, and change type, so that I can efficiently find relevant entries.
- US-05 [P1] As a Compliance Officer, I want to export audit log data as CSV or JSON, so that I can include it in compliance reports and feed it into external SIEM or audit tooling.
- US-06 [P1] As an SRE Manager, I want audit log access controlled by existing PagerDuty roles (Admin/Manager), so that only authorized personnel can view sensitive change history.
- US-07 [P1] As a Platform Engineer, I want audit log entries to capture AI Orchestrations recommendation acceptance events, so that I can track when AI-generated rules were applied and by whom.
- US-08 [P2] As a Security Engineer, I want to receive a notification when a critical EO rule is modified, so that I can proactively review unexpected or unauthorized changes.
- US-09 [P1] As a PM, I want audit log data available via the PagerDuty REST API, so that customers can integrate it into their existing audit and SIEM workflows programmatically.
- US-10 [P1] As a Compliance Officer, I want audit log entries to be retained for a minimum of 12 months (configurable per account tier), so that I can meet common regulatory retention requirements.
- US-11 [P2] As a PM, I want segment analytics events captured for audit log views, exports, and searches, so that I can measure feature adoption and identify usage patterns.
- US-12 [P2] As a User, I want the audit log to surface contextual change history inline within the EO rule editor, so that I can see recent changes without navigating away.
- US-13 [P2] As an Admin, I want the ability to restrict audit log export to specific roles, so that I can prevent unauthorized bulk data extraction.
- US-14 [P1] As a Highly Regulated Customer, I want audit log access scoped to my subdomain, so that data does not co-mingle with other tenants.

---

## Affected Surfaces

- EO Rules write operations (create/update/delete rule, router, global orchestration): US-01, US-02 — every write path must emit an audit event (no existing audit event pattern found in codebase)
- EO persistence layer / database: US-01, US-02, US-10 — new audit_events table required; no current audit schema exists
- EO REST API layer: US-09 — new public endpoint GET /event-orchestrations/audit-logs required
- EO Settings UI: US-04 — new Audit Log view in EO settings page
- EO Rule Editor UI: US-12 — inline history panel within existing rule editor component
- Auth/permissions middleware: US-06, US-13 — extend existing role check middleware for audit log read and export-specific permissions
- Export pipeline: US-05 — no existing CSV/JSON export pattern in EO; pattern exists in other PD services (reference: services/exports/)
- AI Orchestrations event capture: US-07 — hook into recommendation acceptance/dismissal events

## Dependencies & Constraints

- No audit_events table exists in current schema — new table and migration required (found: db/schema.rb, no audit table present)
- No index on actor/timestamp fields currently — will need compound indexes for query performance at scale
- Auth middleware exists for read permissions but has no export-specific role check — extension required (found: middleware/auth/role_check.rb)
- Export pattern exists in other PD services but not in EO — recommend referencing services/exports/csv_export.rb for pattern
- AI Orchestrations recommendation events are emitted but not currently stored as audit records — hook point exists (found: services/ai_orchestrations/recommendation_service.rb)
- Subdomain isolation (US-14) requires account_id scoping on all audit queries — current EO queries use global scope in several places
- Data retention enforcement (US-10) requires a background job for TTL-based deletion — no existing retention job pattern in EO

---

## Success Metrics

- 30% of enterprise-tier EO customers enable audit log within 6 months of GA
- Audit log cited as unblocking factor in at least 5 enterprise deals within first two quarters post-GA
- Mean time to identify root cause for rule-change-related incidents reduced by 40%
- 80%+ of audit log users perform at least one export or API query within 30 days of enablement

## Open Questions

- What is the data volume estimate for audit events at enterprise scale? (needed before storage decisions)
- Should audit events be stored in the primary EO database or a separate audit-specific store?
- What is the exact role mapping for audit log read access — Admin only, or Manager too?

**AI Considerations:** AI components involved — fill in model, data source, evaluation approach, and failure modes before tech design begins. (Relevant to US-07: AI Orchestrations recommendation acceptance/dismissal as audit events.)

---

## Scope Proposal

### EA Scope ✅

Stories shipping to limited EA customers:

- ✅ US-01 — Full change history (core compliance requirement; feature is unusable without this)
- ✅ US-02 — Actor attribution (who changed what; required for incident review use case)
- ✅ US-04 — Filter and search (required to make the log usable; date range + user + orchestration)
- ✅ US-06 — Role-based access control (Admin/Manager read access; security requirement for EA)
- ✅ US-09 — REST API endpoint (programmatic access; required for SIEM integration use case)
- ✅ US-14 — Subdomain scoping (required for regulated-industry EA customers; data isolation)

### GA Scope ⏩

Stories added for general availability:

- ⏩ US-03 — Before/after diff view (high value but not blocking initial compliance use case)
- ⏩ US-05 — CSV/JSON export (needed for GA compliance reporting; export pipeline work)
- ⏩ US-07 — AI Orchestrations events (additive; requires AI team coordination)
- ⏩ US-10 — 12-month retention with configurable tiers (retention policy + background job)
- ⏩ US-08 — Change notifications [P2] (security alerting; additive post-GA)
- ⏩ US-11 — Segment analytics [P2] (instrumentation; additive)
- ⏩ US-12 — Inline rule editor history [P2] (UX enhancement; separate UI work)
- ⏩ US-13 — Export restriction by role [P2] (admin control; additive to export)

### Needs Discussion

(none — all stories resolved)
