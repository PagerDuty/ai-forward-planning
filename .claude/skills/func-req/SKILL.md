---
name: func-req
description: "Use when tech design is reviewed and you need to generate functional requirements. Default (no --milestone) runs EA then GA sequentially. Use --milestone ea|ga only for recovery or partial re-runs."
---

# Functional Requirements Skill

Generate functional requirements docs (one epic + stories per milestone) from the approved PRD and reviewed tech design.

## Arguments

```
/func-req                              # default: runs EA then GA sequentially
/func-req --milestone ea|ga            # recovery/partial: runs one milestone only
```

- `--project` — matches folder under `docs/projects/`. **Optional if `.current-project` is set.**
- `--milestone` — optional. Omit to run both milestones sequentially (happy path). Use `ea` or `ga` only when re-running a specific milestone after a failure or scope change.

## Resolve project name

1. If `--project <name>` is provided: use it. Write the value to `.current-project` at the repo root.
2. Else read `.current-project` from repo root — use the value if present.
3. Else run `git branch --show-current`, strip common prefixes (`feature/`, `fix/`, `feat/`, `bugfix/`) and leading ticket numbers (e.g., `ENG-123-`). If `docs/projects/<slug>/` exists, use it.
4. If none resolve: stop with — *"No project set. Provide `--project <name>` or run `echo '<name>' > .current-project` to set a default."*

When resolved from `.current-project` or git branch (not from `--project`), confirm before proceeding: *"Using project `<name>` — is that right?"*

## Resolve milestone

After resolving the project name, determine which milestones to run:

- **If `--milestone ea` or `--milestone ga` provided:** run that milestone only.
- **If no `--milestone`:** read the Scope Proposal section in `prd.md`.
  - If GA Scope has stories (⏩ items): run EA first, then GA.
  - If GA Scope is empty or absent: run EA only. After EA completes, note: *"No GA stories found in Scope Proposal — skipping GA."*

**Preconditions:**
- `docs/projects/<name>/prd.md` must have `status: scope-finalized` and contain a Scope Proposal section with EA/GA assignments — stop if not met. If `status: approved` but not `scope-finalized`, prompt: "Scope not yet finalized. After the joint meeting, set `status: scope-finalized` in `prd.md` to proceed."
- `docs/projects/<name>/tech-design.md` must have `status: approved` — stop if not met

If `tech-design.md` does not exist:
> "tech-design.md not found. For features with backend/API/DB changes, run `/tech-design --project <name>` first.
> For pure UX/product changes with no backend work, confirm: 'This feature has no API, DB, or backend changes — proceed with user stories only? (yes/no)'. If confirmed, generate a minimal `docs/projects/<name>/tech-design.md` stub automatically with `status: approved` and a note 'No backend changes required — UX-only feature' and proceed."

For the UX-only stub path: technical tasks must be limited to UI work derived from the PRD user stories. Each task should be labeled `[UNVERIFIED]` since there is no tech design to cite. Acceptance criteria should focus on user-facing behaviors only.

---

## Steps

### Step 1: Read all inputs

Read all files before generating anything:

1. `docs/projects/<name>/prd.md` — user stories (section 4), goals (section 2), non-goals (section 3), dependencies (section 6), and the Scope Proposal section (EA/GA assignments). Check `design-mocks` in frontmatter.
2. `docs/projects/<name>/tech-design.md` — API contracts, DB schema, component changes, integration points, open technical questions
3. *(For `--milestone ga` only)* `docs/projects/<name>/functional-requirements-ea.md` if it exists — read to identify what technical tasks were already scoped in EA; do not repeat those tasks in GA

**Fetch design mocks (if present):** If `design-mocks` is set and non-empty in `prd.md` frontmatter, use `mcp__claude_ai_Figma__get_design_context` to fetch the Figma file. Map each screen/frame to its corresponding US-XX story. Hold this mapping — in Step 4, reference specific screen names in acceptance criteria where they add precision (e.g., "matches the empty state shown in the Figma 'No Results' frame"). Do not describe visual styling — reference screen names and behavioral states only.

### Step 2: Select stories for this milestone

From the Scope Proposal section in `prd.md`, extract only the stories assigned to `--milestone` — reference each story by its US-XX ID:
- `ea` → stories listed under **EA Scope** (marked ✅)
- `ga` → stories listed under **GA Scope** (marked ⏩)

Do not mix EA and GA stories in one doc.

If a story is still listed under **Needs Discussion** (marked ⚡) with no clear resolution, stop and ask:
> "Story US-XX is still listed under Needs Discussion in the Scope Proposal section of prd.md with no clear resolution. Which milestone should it be assigned to — EA or GA?"

Do not proceed until ambiguous stories are resolved.

### Step 3: Define the epic

One epic per milestone:
- Title: `[EA] <Project Name>` or `[GA] <Project Name>`
- Goal: what completing this milestone achieves for users (from PRD goals, scoped to this milestone)

### Step 4: Generate stories

For each user story in the milestone (selected by US-XX ID in Step 2), produce a story block with the US-XX ID in the header: `### US-01: <Story Title>`.

**User story** — carried over from the PRD exactly as written (persona + capability + outcome).

**Acceptance criteria** — 2-5 testable, observable behaviors. Derived from:
- PRD: what the user needs to be able to do
- Tech design: specific endpoint responses, UI states, DB constraints where relevant

Where the tech design provides concrete detail, reference it directly in the criterion — e.g., the exact HTTP status code, the specific error message string, the column name or constraint, the component name. Thin criteria ("the endpoint works") are less useful than precise ones ("returns HTTP 400 with `{ "error": "date_from and date_to are required" }` if either date param is absent").

**Technical tasks grouped by work type** — break down the implementation work from the tech design into typed groups. Only include groups that apply to this story:

```
API:
- <specific endpoint to create or modify, from tech design API Contracts section>
- <DB migration if required by this story>

UI:
- <component or page change required>

Background jobs / workers:
- <if applicable>

External dependency:
- <cross-team work required; flag as blocker if it blocks EA>

Misc / launch:
- <docs, runbook, comms, rollout tasks>
```

Rules:
- Every technical task must trace back to the tech design — do not invent tasks
- Each technical task must include a parenthetical citation referencing where it came from in the tech design. Example: `Add GET /audit-logs endpoint (tech-design.md: API Contracts)`. Do not include tasks that cannot be traced to the tech design — if something is missing, update the tech design first.
- If a task type has no work for this story, omit that group entirely
- External dependency tasks must explicitly state the team and what is needed
- Flag any task that is an EA blocker with `⚠️ EA blocker` — remove this flag when the dependency is resolved

### Step 5: Write file

Use `epic-template.md` structure. Write to:
`docs/projects/<name>/functional-requirements-<milestone>.md`

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

**If EA was just written and GA is next (default mode) or `--milestone ga`:** Read `functional-requirements-ea.md` and run:

- **No overlap:** Verify no user story US-XX appears in both EA and GA files
- **No repeated technical tasks:** Verify GA does not repeat technical tasks already in EA (e.g., same endpoint, same DB migration)
- **EA blockers flagged:** Verify all stories with external dependencies are flagged ⚠️ EA blocker if they block EA

If issues are found, fix them inline before prompting for review. Note any fixes in the review prompt.

**If `--milestone ea` only:** Skip this step — GA file doesn't exist yet.

### Step 7: Prompt engineering lead review

**If both milestones were generated (default mode):**
> "Functional requirements drafted:
> - `docs/projects/<name>/functional-requirements-ea.md`
> - `docs/projects/<name>/functional-requirements-ga.md`
>
> Review stories, acceptance criteria, and technical task groupings for both in one pass. Check that external dependencies are correctly flagged. Set `status: approved` on each file when ready, then run `/jira-tickets`."

**If one milestone only (`--milestone ea|ga`):**
> "Functional requirements drafted and saved to `docs/projects/<name>/functional-requirements-<milestone>.md`.
>
> Review the stories, acceptance criteria, and technical task groupings. When ready, set `status: approved` to proceed to `/jira-tickets`."

---

## Notes

- Default (no `--milestone`): runs EA then GA sequentially in one pass. Use `--milestone ea|ga` only for recovery or partial re-runs.
- If GA Scope in the Scope Proposal is empty, GA is skipped automatically — no error, just a note.
- GA stories should be additive over EA — do not repeat what was already built in EA. Read `functional-requirements-ea.md` (Step 1) to identify what was already scoped
- Technical tasks come from the tech design — if the tech design is thin, the task breakdown will be thin; prompt engineering lead to enrich the tech design first
- Misc / launch tasks (docs, runbook, comms) should be included as stories, not as bullet points buried in notes
