# AI-Powered Rule Recommendations for Workflow Engine

**Status:** Draft for review  
**Author:** Product (EO team)  
**Date:** April 2026

---

## Problem

Event Orchestration customers spend significant time manually crafting and tuning routing rules. Most customers start with a blank slate and don't know what rules to create until they've been burned by a missed or misdirected alert. The rules they create are often too broad or too narrow, and they only learn this after an incident.

PagerDuty has rich historical data: years of event patterns, routing decisions, incident outcomes, and resolution paths across thousands of customers. We're not using any of this to help individual customers write better rules. Every customer reinvents the wheel.

## Proposed Solution

Build an AI recommendation engine that analyzes a customer's historical event stream and incident data, and suggests EO rule changes — new rules, modifications to existing rules, threshold adjustments — that would have produced better outcomes.

The recommendations should be:
- Specific and actionable (not "you should add suppression logic" but "add a suppression rule for `service=payments` when `error_type=timeout` fires > 3 times in 5 minutes — here's the rule YAML")
- Explained in plain language ("This rule would have suppressed 47 duplicate alerts last month")
- Easy to accept or dismiss (one-click apply, with the audit log capturing the AI recommendation event)

The ML model should be trained on anonymized, aggregated customer data. We need to be careful about data privacy here — customers should be able to opt out of contributing their data to model training while still receiving recommendations (which would then be based only on their own history).

## User Stories

- As an SRE Manager, I want the system to proactively suggest rule improvements based on my team's historical incident and event patterns, so that I can reduce alert fatigue without spending hours manually analyzing event streams.
- As a Platform Engineer, I want to see a plain-language explanation of why a rule is being recommended and what impact it would have had historically, so that I can make an informed decision about whether to accept it.
- As a Platform Engineer, I want to accept or dismiss AI recommendations with a single click inside the EO rule editor, so that I can act on suggestions without switching contexts.
- As an Admin, I want to control whether my account's event data contributes to model training, so that I can comply with our internal data governance policies.
- As a Compliance Officer, I want AI-generated rule changes captured in the audit log with a flag indicating they were AI-recommended, so that I can distinguish automated changes from human-initiated ones.
- As an SRE Manager, I want to see a history of accepted and dismissed recommendations and their outcomes, so that I can evaluate whether the AI suggestions are improving our incident response.
- As a PM, I want to track recommendation acceptance rate, dismiss rate, and post-acceptance incident reduction, so that I can measure whether the feature is delivering value.

## Non-Goals

- This feature does not automatically apply rules without human review — all recommendations require explicit acceptance
- We are not building a general-purpose ML platform; the model is purpose-built for EO rule optimization
- We are not recommending changes to incident response workflows outside of EO

## Success Metrics

- 30% of EA customers accept at least one AI recommendation within 60 days of enablement
- Customers who accept recommendations see a measurable reduction in alert volume (target: 15% reduction in duplicate alerts)
- Recommendation acceptance rate > 25% (measures quality of recommendations)
- Audit log adoption increases as a side effect (customers enable audit log to track AI-applied changes)

## Open Questions

- Which model architecture are we using? Need ML team input on whether we fine-tune an existing model or train from scratch.
- What's the minimum data history required to generate useful recommendations? (Hypothesis: 90 days of event data)
- How do we handle customers with very low event volume who don't have enough history?
- How do we evaluate recommendation quality before shipping to customers?
- Privacy review needed: what exactly is anonymized before data is used for training?
