---
name: jira-tickets
description: "Use when functional requirements are ready and approved. Creates JIRA Epics and Stories for EA then GA sequentially using JIRA MCP."
---

# JIRA Tickets Skill

Create JIRA Epics and Stories from approved functional requirements docs using JIRA MCP.

## Arguments

```
/jira-tickets --project <name>
```

- `--project` — matches folder under `docs/projects/`. **Optional if `.current-project` is set.**

**Resolve project:** `--project <name>` → write to `.current-project`. Else read `.current-project`. Else derive from git branch (strip `feature/`, `fix/`, `feat/`, `bugfix/`, leading ticket numbers like `ENG-123-`). If resolved from context, confirm first. If unresolvable, stop.

**Precondition:** Check which func-req files exist and have `status: approved`:
- If `functional-requirements-ea.md` exists and is approved: run EA.
- If `functional-requirements-ga.md` exists and is approved: run GA after EA.
- If only EA exists and is approved: run EA only, note: *"No approved GA functional requirements found — skipping GA."*
- If neither exists or neither is approved: stop with — *"No approved functional-requirements files found. Run `/func-req` first."*

---

## Steps

### Step 1: Read functional requirements

Read `docs/projects/<name>/functional-requirements-<milestone>.md` and `docs/projects/<name>/prd.md` in full. The PRD is needed for the human-readable feature name for the Epic summary.

**Terminal status check:** If the functional requirements file has `status: done` AND all stories already have JIRA keys written back, stop:
> "Tickets already created for this milestone (status: done). To add new stories, add them to the functional requirements file without JIRA keys and re-run — existing stories will be skipped."

If `status: done` but some stories are missing JIRA keys (partial run), continue — skip stories that already have keys, create only the missing ones.

Note all technical task groups per story — each group becomes one or more JIRA stories. Note any tasks flagged as `⚠️ EA blocker`.

### Step 2: Confirm JIRA project, team prefix, and label

Check if `docs/projects/<name>/.jira-config.md` exists.

**If it exists**, read the stored values and show:
> "Using saved JIRA config: project `<jira-project-key>`, base prefix `<team-base-prefix>`, label `<project-label>`. Press enter to confirm or provide corrections."

**If it does not exist**, ask:
> "Three quick things before I create tickets:
> 1. JIRA project key? (e.g., `PROJ`, `ALERT`)
> 2. Team/area base prefix? (e.g., `WE`, `ALERT`) — I'll append the work-type suffix (`-API`, `-UI`, `-DB`, `-OPS`) per story
> 3. Project label to apply to all stories? (e.g., `Event_Enrichment`, `Q2_2026`)"

After the user answers, write `docs/projects/<name>/.jira-config.md`:

```
---
jira-project-key: <value>
team-base-prefix: <value>
project-label: <value>
---
```

### Step 3: Create the Epic

Use JIRA MCP `createJiraIssue` with:
- **Issue type:** Epic
- **Summary:** `<EA|GA>: <Human-readable feature name>` — use the full feature name, not the slug
- **Description:**
  ```
  <One paragraph describing the feature scope for this milestone.>

  ## Resources
  - Functional Requirements: docs/projects/<name>/functional-requirements-<milestone>.md
  - PRD: docs/projects/<name>/prd.md
  - Tech Design: docs/projects/<name>/tech-design.md
  ```

Record the returned Epic key (e.g., `PROJ-42`).

### Step 4: Create Stories — one per technical task, sized to ~3 days

For each technical task group, create **one JIRA Story per task**. Apply sizing judgment:

- **Target: ~3 days of work per story.**
- **If a task is clearly larger than 3 days**: split into two stories before creating.
- **If multiple tasks are each trivially small (<1 day)** and closely related: combine them into one story.
- Add an `// est: ~N days` comment under Implementation Details so estimates are visible after delivery.

**Summary format:** `[<TEAM-AREA>] <Feature>: <Specific Task>`

Work-type suffixes:
- API / backend → `<base>-API` (e.g., `WE-API`)
- UI / frontend → `<base>-UI` (e.g., `WE-UI`)
- DB / migration → `<base>-DB` (e.g., `WE-DB`)
- Ops / launch → `<base>-OPS` (e.g., `WE-OPS`)

**Description format:**
```markdown
## End Goal

<One paragraph describing the concrete deliverable for this story.>

## Implementation Details

<Specific technical steps: file paths, component names, API contracts, patterns to follow.>

// est: ~N days

## Out of Scope

- <Explicit exclusions — what is intentionally NOT done in this story>
```

**Epic Link:** set to the Epic key from Step 3

**Labels:** the project label from Step 2

### Step 5: Create dependency links for EA blockers

For any story flagged `⚠️ EA blocker`:
- Use JIRA MCP `createIssueLink` with link type `"Depends on"` between the blocked story and the external dependency
- If the external dependency has no JIRA key yet, note it in the story description and flag for the engineering lead to link manually

### Step 6: Write keys back to file

Update `docs/projects/<name>/functional-requirements-<milestone>.md`:
- Set `jira-epic-key: <EPIC-KEY>` in frontmatter
- Add JIRA issue key next to each technical task:
  ```
  API:
  - Add GET /audit-logs endpoint [PROJ-43]

  UI:
  - Date range filter component [PROJ-45]
  ```

### Step 7: Confirm completion

Set `status: done` on the func-req file, then:

> "Created Epic <EPIC-KEY> and <N> stories in JIRA project <PROJECT>.
> EA blockers with dependency links: <N>.
> Issue keys written back to `docs/projects/<name>/functional-requirements-<milestone>.md`.
>
> **Story count: <N> stories @ ~3 days each = ~<N×3> engineer-days estimated.**"

---

## Notes

- JIRA MCP must be configured and authenticated before running
- One story per technical task, sized to ~3 days — split larger tasks, combine trivially small ones
- Story descriptions are technical and specific — include file paths, component names, API contracts from the tech design
- If a story already has a JIRA key written back to the file, skip it — this skill is idempotent
- If a JIRA MCP call fails mid-run, stop and report partial state. Do not retry automatically — confirm with the engineering lead before resuming
- The `team-base-prefix` in `.jira-config.md` is the base; work-type suffixes are appended per story during creation
