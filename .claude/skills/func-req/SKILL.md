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

**Resolve project:** `--project <name>` → write to `.current-project`. Else read `.current-project`. Else derive from git branch (strip `feature/`, `fix/`, `feat/`, `bugfix/`, leading ticket numbers like `ENG-123-`). If resolved from context, confirm first. If unresolvable, stop.

## Preconditions

- `docs/projects/<name>/prd.md` must have `status: approved` AND a Scope Proposal section with EA/GA assignments (no remaining ⚡ items). If ⚡ items remain, stop: *"Scope not finalized — resolve the ⚡ items in the Scope Proposal section of prd.md before running func-req."*
- `docs/projects/<name>/tech-design.md` must have `status: approved` — stop if not met.

If `tech-design.md` does not exist:
> "tech-design.md not found. For features with backend/API/DB changes, run `/tech-design --project <name>` first.
> For pure UX/product changes with no backend work, confirm: 'This feature has no API, DB, or backend changes — proceed with user stories only? (yes/no)'. If confirmed, generate a minimal `docs/projects/<name>/tech-design.md` stub with `status: approved` and note 'No backend changes required — UX-only feature' and proceed."

For the UX-only stub path: technical tasks must be limited to UI work derived from PRD user stories. Each task labeled `[UNVERIFIED]`. Acceptance criteria focus on user-facing behaviors only.

---

## Steps

### Step 1: Read all inputs

Read all files before generating anything:

1. `docs/projects/<name>/prd.md` — user stories (section 4), goals (section 2), non-goals (section 3), dependencies (section 6), and the Scope Proposal section (EA/GA assignments). Check `design-mocks` in frontmatter.
2. `docs/projects/<name>/tech-design.md` — API contracts, DB schema, component changes, integration points, open technical questions
3. `docs/projects/<name>/functional-requirements-ea.md` if it exists — read to identify what technical tasks were already scoped in EA; do not repeat those tasks in GA

**Fetch design mocks (if present):** If `design-mocks` is set in `prd.md`, use `mcp__claude_ai_Figma__get_design_context`. Map each screen/frame to its corresponding US-XX story. In Step 4, reference specific screen names in acceptance criteria where they add precision (e.g., "matches the empty state shown in the Figma 'No Results' frame"). Reference screen names and behavioral states only — not visual styling.

### Step 2: Determine milestones to run

Read the Scope Proposal section in `prd.md`:
- If GA Scope has stories (⏩ items): run EA first, then GA.
- If GA Scope is empty or absent: run EA only. After EA completes, note: *"No GA stories found in Scope Proposal — skipping GA."*

If a story is still listed under Needs Discussion (⚡) with no resolution, stop:
> "Story US-XX is still listed under Needs Discussion in the Scope Proposal with no resolution. Move it to EA or GA before running func-req."

### Step 3: Define the epic

One epic per milestone:
- Title: `[EA] <Project Name>` or `[GA] <Project Name>`
- Goal: what completing this milestone achieves for users (from PRD goals, scoped to this milestone)

### Step 4: Generate stories

For each user story in the milestone (by US-XX ID from the Scope Proposal), produce a story block:

```markdown
### US-01: <Story Title>

**As a** <persona>, **I want** <capability> **so that** <outcome>.

**Acceptance Criteria:**
- [ ] <User-facing, observable behavior>
- [ ] <Reference specific endpoint/component/state from tech design where relevant>

**Technical Tasks:**

API:
- <Endpoint to create or modify — method, path, key params (tech-design.md: API Contracts)>
- <DB migration if required (tech-design.md: DB Schema)>

UI:
- <Component or page change (tech-design.md: Component Changes)>

Background jobs / workers:
- <Worker or job description (tech-design.md: section)>

External dependency:
- <Team name>: <what is needed> ⚠️ EA blocker *(remove tag if not a blocker)*

Misc / launch:
- <Docs, runbook, comms, rollout task>
```

Rules:
- Include only task groups that apply — omit empty groups entirely
- Every technical task must trace back to the tech design and include a parenthetical citation: `Add GET /audit-logs endpoint (tech-design.md: API Contracts)`. Tasks that cannot be cited must be flagged `[UNVERIFIED]`
- Acceptance criteria: 2-5 testable behaviors. Reference exact HTTP status codes, error message strings, column names, component names where the tech design provides them. Thin criteria ("the endpoint works") are less useful than precise ones.
- External dependency tasks must explicitly state the team and what is needed
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

### Step 6: Consistency check

After both files are written:
- **No overlap:** Verify no US-XX story appears in both EA and GA files
- **No repeated tasks:** Verify GA does not repeat technical tasks already in EA (same endpoint, same migration)
- **EA blockers flagged:** Verify all stories with external dependencies are flagged ⚠️ EA blocker if they block EA

Fix any issues inline before prompting for review. Note any fixes in the review prompt.

### Step 7: Prompt engineering lead

> "Functional requirements drafted:
> - `docs/projects/<name>/functional-requirements-ea.md`
> - `docs/projects/<name>/functional-requirements-ga.md` *(if GA exists)*
>
> Review stories, acceptance criteria, and technical task groupings. Check that external dependencies are correctly flagged. Set `status: approved` on each file when ready, then run `/jira-tickets`."

---

## Notes

- GA stories are additive over EA — do not repeat what was already built. Read `functional-requirements-ea.md` (Step 1) to identify what was already scoped.
- If GA Scope in the Scope Proposal is empty, GA is skipped automatically.
- Technical tasks come from the tech design — if the tech design is thin, the task breakdown will be thin. Prompt eng lead to enrich the tech design first.
- Misc / launch tasks (docs, runbook, comms) should be included as stories, not buried in notes.
- If func-req files already exist with `status: done`, stop: *"Tickets already created. To add new stories, add them to the functional requirements file without JIRA keys and re-run — existing stories will be skipped."*
