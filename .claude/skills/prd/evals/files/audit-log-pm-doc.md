**Event Orchestration Audit Log**

# **Context**

Event Orchestration (EO) is PagerDuty’s rules engine that allows customers to route, suppress, enrich, and automate actions on incoming events before they become incidents. Enterprise customers increasingly rely on EO to encode critical operational logic—routing rules, suppression policies, and escalation workflows—directly into PagerDuty.

However, as EO rule sets grow in complexity and more teams contribute to configuration changes, customers lack any visibility into who changed what, when, and why. This absence of an audit trail creates significant friction for large enterprise organizations that are subject to internal governance requirements, change management processes, and external compliance mandates.

Without an audit log, EO adoption at scale is blocked for a substantial segment of enterprise and regulated-industry customers. Providing audit log capabilities is widely considered “table stakes” for any enterprise-grade rules or automation system.

# **Persona**

## **Buyer Persona**

**Titles:** CISO, VP of Engineering, Head of IT Operations, Head of Compliance, CIO

Buyers in this space are primarily motivated by risk reduction and regulatory compliance. They need demonstrable proof that their operational tooling meets internal audit and governance standards before they can approve broad deployment. Key drivers:

* Ability to demonstrate change traceability to internal audit teams

* Compliance with regulatory frameworks (SOC 2, ISO 27001, HIPAA, FedRAMP)

* Confidence to expand EO usage across more services and teams without loss of control

## **User Persona**

**Titles:** SRE Manager, NOC Manager, Platform Engineer, Security Engineer, Compliance Officer

Primary users are the operators and managers responsible for maintaining EO configuration integrity. They need to understand the change history of rules to debug unexpected behavior, attribute configuration changes to specific individuals, and demonstrate compliance during audits. Key needs:

* Quickly identify who last modified a rule and what changed

* Correlate a production incident or misbehaving rule with a recent configuration change

* Produce audit reports for compliance reviews without manual data extraction

* Receive alerts when critical rules are modified unexpectedly

# **Problem**

PagerDuty’s Event Orchestration currently provides no mechanism to record, query, or export a history of changes made to orchestration rules, routers, or global configurations. This creates three compounding problems:

## **1\. Enterprise Adoption is Blocked**

Large enterprise customers—particularly those in financial services, healthcare, and government—require all operational tooling to meet change management and audit standards. Without an audit log, EO fails procurement and security reviews, preventing deployment at scale. Sales teams report this as one of the top reasons enterprise deals stall or require workarounds.

## **2\. Operational Risk is Elevated**

When an EO rule change causes unexpected behavior—such as suppressing alerts that should have triggered an incident—there is currently no way to identify what changed, who changed it, or when. This makes root cause analysis slow and difficult, and increases the blast radius of misconfiguration.

## **3\. Feature Velocity is Constrained**

Customers who are aware of this gap adopt a “freezing” pattern where they limit who can edit EO rules and move slowly to avoid untracked changes. This reduces the perceived value of AI Orchestrations recommendations and other EO features, since customers are reluctant to apply changes they cannot audit.

# **Value Proposition**

An EO Audit Log gives enterprise customers the traceability they need to confidently deploy and evolve Event Orchestration at scale. By capturing a structured, queryable record of every change to EO configuration—including the actor, timestamp, change type, and before/after state—PagerDuty can:

* **Unblock enterprise sales:** Satisfy audit and compliance requirements that currently prevent EO deployment in regulated industries.

* **Reduce operational risk:** Enable fast root cause analysis when a rule change contributes to an incident, by surfacing exactly what changed and when.

* **Build customer trust:** Give platform and security teams confidence to delegate EO management to broader teams, increasing adoption depth.

* **Complement AI Orchestrations:** Provide the traceability layer that makes customers comfortable accepting and applying AI-generated rule recommendations at scale.

**Key Metrics:**

* Reduction in blocked enterprise deals citing audit/compliance gaps

* Increase in EO adoption rate among enterprise-tier customers

* Decrease in mean time to identify root cause for rule-change-related incidents

* Audit log query volume as a proxy for active compliance use

* AI Orchestrations recommendation acceptance rate (expected to increase as trust grows)

# **Features & Requirements**

## **High Level User Stories**

| High Level User Story | Priority |
| :---- | :---: |
| As a Compliance Officer, I want a full history of changes made to EO rules, routers, and global orchestrations, so that I can satisfy internal and external audit requirements. | P1 |
| As an SRE Manager, I want to see who made a change to a specific rule and when, so that I can attribute configuration changes to individuals during an incident review. | P1 |
| As a Platform Engineer, I want to view the before and after state of a rule change, so that I can quickly understand what was modified and revert if necessary. | P1 |
| As a Platform Engineer, I want to filter and search the audit log by date range, user, orchestration, and change type, so that I can efficiently find relevant entries. | P1 |
| As a Compliance Officer, I want to export audit log data as CSV or JSON, so that I can include it in compliance reports and feed it into external SIEM or audit tooling. | P1 |
| As an SRE Manager, I want audit log access controlled by existing PagerDuty roles (Admin/Manager), so that only authorized personnel can view sensitive change history. | P1 |
| As a Platform Engineer, I want audit log entries to capture AI Orchestrations recommendation acceptance events, so that I can track when AI-generated rules were applied and by whom. | P1 |
| As a Security Engineer, I want to receive a notification when a critical EO rule is modified, so that I can proactively review unexpected or unauthorized changes. | P2 |
| As a PM, I want audit log data available via the PagerDuty REST API, so that customers can integrate it into their existing audit and SIEM workflows programmatically. | P1 |
| As a Compliance Officer, I want audit log entries to be retained for a minimum of 12 months (configurable per account tier), so that I can meet common regulatory retention requirements. | P1 |
| As a PM, I want segment analytics events captured for audit log views, exports, and searches, so that I can measure feature adoption and identify usage patterns. | P2 |
| As a User, I want the audit log to surface contextual change history inline within the EO rule editor, so that I can see recent changes without navigating away. | P2 |
| As an Admin, I want the ability to restrict audit log export to specific roles, so that I can prevent unauthorized bulk data extraction. | P2 |
| As a Highly Regulated Customer, I want audit log access scoped to my subdomain, so that data does not co-mingle with other tenants. | P1 |

# **Solution**

The solution will be delivered in phased milestones to validate demand and incrementally deliver value. Milestone scope is provisional pending technical discovery.

## **Milestone 1: Discovery & Technical Scoping**

* Audit existing EO backend services to identify all write operations that require logging

* Define the audit event schema (actor, timestamp, resource type, resource ID, action, before state, after state)

* Assess data volume and storage requirements for log retention at enterprise scale

* Identify role/permission model for audit log access

* Assess feasibility of inline contextual history in the EO rule editor UI

## **Milestone 2: Backend Audit Log Infrastructure (EA)**

* Implement audit event capture for all EO write operations (create, update, delete) on rules, routers, and global orchestrations

* Capture AI Orchestrations recommendation acceptance/dismissal as audit events

* Implement secure, queryable storage with configurable retention (default 12 months)

* Internal API endpoint for querying audit log by resource, actor, date range, and action type

## **Milestone 3: In-Product Audit Log UI & Export (EA → GA)**

* Audit Log view in EO settings: filterable by date, user, orchestration, change type

* Diff view showing before/after state of rule changes

* CSV and JSON export for offline reporting

* Public REST API endpoint for programmatic audit log access

* Role-based access controls (Admin/Manager read access, configurable export restrictions)

## **Milestone 4: GA Hardening & Notifications**

* Change notification alerts for critical rule modifications (configurable by user/team)

* Inline audit history surfaced contextually within the EO rule editor

* Product limits, entitlements, and tier-based retention configuration

* SIEM integration guidance and documentation

* GA activities: code cleanup, feature flag removal, updated API docs, Segment events

# **Designs**

Designs TBD — to be initiated following Milestone 1 technical scoping.

# **Related Documentation**

* AI Orchestrations: PRD

* Event Orchestration API Documentation

* PagerDuty Role-Based Access Control Documentation

* PagerDuty SOC 2 Compliance Documentation

* Customer feedback synthesis: EO Audit Log requests (to be linked post-discovery)

# **Success Metrics**

* 30% of enterprise-tier EO customers enable audit log within 6 months of GA

* Audit log cited as unblocking factor in at least 5 enterprise deals within first two quarters post-GA

* Mean time to identify root cause for rule-change-related incidents reduced by 40% for customers using audit log

* 80%+ of audit log users perform at least one export or API query within 30 days of enablement

* 5% increase in AI Orchestrations recommendation acceptance rate attributable to trust improvements from audit log

# **Future Considerations**

The EO Audit Log infrastructure will establish foundational capabilities that enable a broader set of governance and automation features:

* Automated rollback: allow users to revert to a previous rule state directly from audit log history

* Change approval workflows: require manager approval before EO rule changes are applied (change management gate)

* Scheduled audit log digest emails for compliance officers

* Cross-product audit correlation: link EO audit events to PagerDuty Incident and service change events for end-to-end traceability

* Terraform / IaC integration: surface audit events in Terraform plan output to show drift between code and live configuration

* AI anomaly detection on audit log: flag unusual change patterns (e.g., bulk rule deletions, off-hours changes) as potential security events