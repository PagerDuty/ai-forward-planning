---
name: jira-tickets
description: "Use when functional requirements are ready and approved, and you need to create JIRA Epics and Stories using JIRA MCP. Default (no --milestone) runs EA then GA sequentially. Use --milestone ea|ga only for recovery or partial re-runs."
---

# JIRA Tickets Skill

Create JIRA Epics and Stories from approved functional requirements docs using JIRA MCP.

## Arguments

```
/jira-tickets                              # default: runs EA then GA sequentially
/jira-tickets --milestone ea|ga            # recovery/partial: runs one milestone only
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

- **If `--milestone ea` or `--milestone ga` provided:** run that milestone only.
- **If no `--milestone`:** check which func-req files exist and have `status: approved`:
  - If `functional-requirements-ea.md` exists and is approved: run EA.
  - If `functional-requirements-ga.md` exists and is approved: run GA after EA.
  - If only EA exists: run EA only, note: *"No approved GA functional requirements found — skipping GA."*
  - If neither exists: stop with — *"No approved functional-requirements files found. Run `/func-req` first."*

**Precondition:** Each milestone's `functional-requirements-<milestone>.md` must have `status: approved` before that milestone is processed.

---

## Steps

### Step 1: Read functional requirements

Read `docs/projects/<name>/functional-requirements-<milestone>.md` and `docs/projects/<name>/prd.md` in full. The PRD is needed to get the human-readable feature name for the Epic summary.

**Terminal status check:** If the functional requirements file has `status: done` AND all stories already have JIRA keys written back, stop with:
> "Tickets already created for this milestone (status: done). To add new stories, add them to the functional requirements file without JIRA keys and re-run — existing stories will be skipped."

If `status: done` but some stories are missing JIRA keys (partial run), continue with partial run recovery — skip stories that already have keys, create only the missing ones.

Note all technical task groups per story — each group becomes one or more JIRA stories.
Note any tasks flagged as `⚠️ EA blocker` — these need dependency links.

### Step 2: Confirm JIRA project, team prefix, and label

Check if `docs/projects/<name>/.jira-config.md` exists.

**If it exists**, read the stored values and show:
> "Using saved JIRA config: project `<jira-project-key>`, base prefix `<team-base-prefix>`, label `<project-label>`. Press enter to confirm or provide corrections."

Use the stored values unless the user provides corrections.

**If it does not exist**, ask:
> "Three quick things before I create tickets:
> 1. JIRA project key? (e.g., `PROJ`, `ALERT`)
> 2. Team/area base prefix? (e.g., `WE`, `ALERT`) — I'll append the work-type suffix (`-API`, `-UI`, `-DB`, `-OPS`) per story
> 3. Project label to apply to all stories? (e.g., `Event_Enrichment`, `Q2_2026`) — used for filtering on the board"

After the user answers, write `docs/projects/<name>/.jira-config.md` with the values:

```
---
jira-project-key: <value>
team-base-prefix: <value>
project-label: <value>
---
```

This config is read automatically on subsequent runs (e.g., EA then GA, or re-runs).

### Step 3: Create the Epic

Use JIRA MCP `createJiraIssue` with:
- **Issue type:** Epic
- **Summary:** `<EA|GA>: <Human-readable feature name>` — use the feature's full name, not the `--project` slug (e.g., if `--project audit-log`, summary is `EA: Audit Log: Compliance Access`, not `EA: audit-log`)
- **Description:** Narrative paragraph describing what this epic covers + link to the functional requirements doc:
  ```
  <One paragraph describing the feature scope for this milestone.>

  ## Resources
  - Functional Requirements: docs/projects/<name>/functional-requirements-<milestone>.md
  - PRD: docs/projects/<name>/prd.md
  - Tech Design: docs/projects/<name>/tech-design.md
  ```

Record the returned Epic key (e.g., `PROJ-42`).

### Step 4: Create Stories — one per technical task, sized to ~3 days

For each technical task group in the functional requirements, create **one JIRA Story per task**. Before creating, apply sizing judgment:

- **Target: ~3 days of work per story.** This is the team's baseline chunk size.
- **If a task is clearly larger than 3 days** (e.g., "Build the full audit log API"): split it into two stories before creating. Use your judgment on the natural seam.
- **If multiple tasks are each trivially small (<1 day)** and closely related: combine them into one story.
- Add an `// est: ~N days` comment in the story description under Implementation Details so estimates are visible and comparable after delivery.

Break each task group into individual stories:

**Summary format:** `[<TEAM-AREA>] <Feature>: <Specific Task>`

The `<TEAM-AREA>` is the base prefix with a work-type suffix appended:
- API / backend tasks → `<base>-API` (e.g., `WE-API`)
- UI / frontend tasks → `<base>-UI` (e.g., `WE-UI`)
- DB / migration tasks → `<base>-DB` (e.g., `WE-DB`)
- Ops / launch tasks → `<base>-OPS` (e.g., `WE-OPS`)

Examples:
- `[API] Audit Log: Add GET /audit-logs endpoint with date range params`
- `[UI] Audit Log: Date range filter component`
- `[DB] Audit Log: Add user_id index to events table`
- `[OPS] Audit Log: Write runbook for log retention policy`

**Description format (free-form markdown):**
```markdown
## End Goal

<One paragraph describing what this specific story achieves — the concrete deliverable.>

## Implementation Details

<Specific technical steps: file paths, component names, API contracts, branch to build on, 
patterns to follow or reference. Be specific — engineers should be able to start from this.>

## Out of Scope

- <Explicit exclusions — what is intentionally NOT done in this story>
```

**Epic Link:** set to the Epic key from Step 3

**Labels:** the project label confirmed in Step 2

### Step 5: Create dependency links for EA blockers

For any story flagged `⚠️ EA blocker` in the functional requirements:
- Use JIRA MCP `createIssueLink` with link type `"Depends on"` between the blocked story and the external dependency
- If the external dependency doesn't have a JIRA key yet, note it in the story description under Implementation Details and flag for the engineering lead to link manually

### Step 6: Write keys back to file

Update `docs/projects/<name>/functional-requirements-<milestone>.md`:
- Set `jira-epic-key: <EPIC-KEY>` in frontmatter
- Add JIRA issue key next to each technical task in the relevant story block:
  ```
  API:
  - Add GET /audit-logs endpoint [PROJ-43]
  - Add user_id index to events table [PROJ-44]
  
  UI:
  - Date range filter component [PROJ-45]
  ```

### Step 7: Confirm completion

After writing keys back to the file, also update the frontmatter of `docs/projects/<name>/functional-requirements-<milestone>.md` to set `status: done`.

> "Created Epic <EPIC-KEY> and <N> stories in JIRA project <PROJECT>.
> EA blockers with dependency links: <N>.
> Issue keys written back to `docs/projects/<name>/functional-requirements-<milestone>.md`.
> Status set to `done`.
>
> **Story count: <N> stories @ ~3 days each = ~<N×3> engineer-days estimated.**
> Track actual delivery time per story to measure AI planning impact over time."

---

## Notes

- Default (no `--milestone`): runs EA then GA in one pass. GA is skipped automatically if no approved GA func-req file exists. Use `--milestone ea|ga` only for recovery.
- JIRA MCP must be configured and authenticated before running
- **One story per technical task, sized to ~3 days** — split tasks that are clearly larger, combine tasks that are trivially small. Story count per user story will vary by surface complexity.
- Story descriptions are technical and specific — include file paths, component names, API contracts where known from the tech design
- If a story already has a JIRA key written back to the file, skip it — this skill is idempotent
- If a JIRA MCP call fails mid-run, stop and report partial state (e.g., "Epic PROJ-42 created, 3 of 9 stories created"). Do not retry automatically — confirm with the engineering lead before resuming
- If `status: done`, the skill will warn and stop on re-run unless stories are missing JIRA keys (partial run recovery). Do not re-run to create duplicates.
- The `team-base-prefix` in `.jira-config.md` is the base (e.g., `WE`); work-type suffixes (`-API`, `-UI`, `-DB`, `-OPS`) are appended per story during ticket creation.
