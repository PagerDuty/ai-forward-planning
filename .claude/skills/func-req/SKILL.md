---
name: func-req
description: "Use when tech design is reviewed and approved. Generates functional requirements docs (EA then GA) from the approved PRD and tech design. Runs both milestones sequentially in one pass."
---

# Functional Requirements Skill

Generate functional requirements docs (one epic + stories per milestone) from the approved PRD and tech design.

## Arguments

```
/func-req --project <name>
```

- `--project` — matches folder under `docs/projects/`. **Optional if `.current-project` is set.**

**Resolve project:** Use `--project <name>` if provided (writes to `.current-project`). Else read `.current-project`. If neither, stop: *"No project set. Run with `--project <name>` or set `.current-project`."*

## Preconditions

- `docs/projects/<name>/prd.md` must have `status: approved` AND a Scope Proposal section with EA/GA assignments (no remaining ⚡ items). If ⚡ items remain, stop: *"Scope not finalized — resolve the ⚡ items in the Scope Proposal section of prd.md before running func-req."*
- `docs/projects/<name>/tech-design.md` must have `status: approved` — stop if not met.

If `tech-design.md` does not exist: *"tech-design.md not found. For UI-only features with no backend changes, confirm before proceeding — tasks will be labeled `[UNVERIFIED]`. Otherwise run `/tech-design` first."*

---

## Steps

### Step 1: Read all inputs

Read all files before generating anything:

1. `docs/projects/<name>/prd.md` — user stories (section 4), goals (section 2), non-goals (section 3), dependencies (section 6), and the Scope Proposal section (EA/GA assignments). Check `design-mocks` in frontmatter.
2. `docs/projects/<name>/tech-design.md` — API contracts, DB schema, component changes, integration points, open technical questions
3. `docs/projects/<name>/functional-requirements-ea.md` if it exists — read to identify what was already scoped in EA; GA must not repeat those tasks

**Fetch design mocks (if present):** If `design-mocks` is set, fetch via Figma MCP. Map each screen to its US-XX story. Reference specific screen names in acceptance criteria where they add precision — behavioral states only, not visual styling.

### Step 2: Determine milestones and select stories

Read the Scope Proposal section in `prd.md`:
- If GA Scope has stories (⏩ items): run EA first, then GA.
- If GA Scope is empty: run EA only — note: *"No GA stories found — skipping GA."*
- If any story is still ⚡ with no resolution: stop and ask which milestone it belongs to.

**Story selection is strict:** EA file contains only ✅ stories. GA file contains only ⏩ stories. No story appears in both files.

### Step 3: Define the epic

One epic per milestone:
- Title: `[EA] <Project Name>` or `[GA] <Project Name>`
- Goal: what completing this milestone achieves for users (from PRD goals, scoped to this milestone)

### Step 4: Generate stories

For each user story in the milestone (by US-XX ID), produce a story block:

```markdown
### US-01: <Story Title>

**As a** <persona>, **I want** <capability> **so that** <outcome>.

**Acceptance Criteria:**
- [ ] <User-facing, observable behavior>
- [ ] <Reference specific endpoint/component/state from tech design where relevant>

**Technical Tasks:**

API:
- <Endpoint to create or modify — method, path, key params (tech-design.md: API Contracts)>

UI:
- <Component or page change (tech-design.md: Components)>

Background jobs / workers:
- <Worker or job (tech-design.md: section)>

External dependency:
- <Team name>: <what is needed> ⚠️ EA blocker *(remove tag if not a blocker)*

Misc / launch:
- <Docs, runbook, comms, rollout task>
```

Rules:
- Include only task groups that apply — omit empty groups
- Every task must trace back to the tech design with a citation: `Add GET /audit-logs endpoint (tech-design.md: API Contracts)`. Uncited tasks must be flagged `[UNVERIFIED]`
- Acceptance criteria: 2-5 testable behaviors. Be precise — reference exact status codes, error strings, column names where the tech design provides them
- Flag any task that blocks EA with `⚠️ EA blocker`

### Step 5: Write files

Write to:
- `docs/projects/<name>/functional-requirements-ea.md`
- `docs/projects/<name>/functional-requirements-ga.md` (if GA scope exists)

Frontmatter:
```yaml
---
project: <name>
milestone: <ea|ga>
created: <today's date YYYY-MM-DD>
status: draft
jira-epic-key:
---
```

### Step 6: Prompt engineering lead

> "Functional requirements drafted:
> - `docs/projects/<name>/functional-requirements-ea.md`
> - `docs/projects/<name>/functional-requirements-ga.md` *(if GA exists)*
>
> Review stories, acceptance criteria, and technical task groupings. Set `status: approved` on each file when ready, then run `/jira-tickets`."

---

## Notes

- GA stories are additive over EA — do not repeat what was already scoped in EA (Step 1)
- Technical tasks come from the tech design — if the tech design is thin, the task breakdown will be thin; ask eng lead to enrich it first
- Misc / launch tasks (docs, runbook, comms) should be included as stories, not buried in notes
