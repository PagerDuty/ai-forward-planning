---
project: {{project-name}}
milestone: {{ea|ga}}
created: {{date}}
status: draft
jira-epic-key:
---

# Epic: [{{EA|GA}}] {{Project Name}}

**Milestone:** {{EA | GA}}
**Goal:** {{One sentence: what does completing this milestone achieve for users?}}
**Sources:** `prd.md` · `prd.md (Scope Proposal section)` · `tech-design.md`

---

## Stories

<!-- Include only groups that apply — omit the rest entirely -->
### US-01: {{Story Title}}

**As a** {{persona}}, **I want** {{capability}} **so that** {{outcome}}.

**Acceptance Criteria:**
- [ ] {{User-facing, observable behavior}}
- [ ] {{User-facing, observable behavior}}
- [ ] {{Reference specific endpoint/component/state from tech design where relevant}}

**Technical Tasks:**

API:
- {{Endpoint to create or modify — method, path, key params (tech-design.md: API Contracts)}}
- {{DB migration if required (tech-design.md: DB Schema)}}

UI:
- {{Component or page change (tech-design.md: Component Changes)}}

Background jobs / workers:
- {{Worker or job description (tech-design.md: section)}}

External dependency:
- {{Team name}}: {{what is needed}} ⚠️ EA blocker *(remove tag if not a blocker)*

Misc / launch:
- {{Docs, runbook, comms, rollout task}}

---

### US-02: {{Story Title}}

**As a** {{persona}}, **I want** {{capability}} **so that** {{outcome}}.

**Acceptance Criteria:**
- [ ] {{Criterion}}
- [ ] {{Criterion}}

**Technical Tasks:**

API:
- {{Task (tech-design.md: API Contracts)}}

---

<!-- Repeat as needed. Each story maps to one user story from prd.md (Scope Proposal section) -->
### US-03: {{Story Title}}

---

<!-- 
  Rules:
  - One story per US-XX user story from prd.md (Scope Proposal section) assigned to this milestone. Story IDs must match the PRD exactly.
  - Omit technical task groups that don't apply to a story
  - GA stories are additive — don't repeat EA work
  - Misc/launch tasks are stories too, not footnotes
  - Every technical task must include a citation to the tech design section it came from
  - Tasks that cannot be cited must be flagged [UNVERIFIED]
-->
