---
name: jira-tickets
description: "Use when functional requirements are ready and approved, and it's time to create JIRA tickets. Creates Epics and Stories for Early Access (EA) then General Availability (GA) milestones sequentially via JIRA MCP. Handles backlog creation, story sizing, and writing issue keys back to requirements docs."
---

# JIRA Tickets Skill

Create JIRA Epics and Stories from approved functional requirements docs using JIRA MCP. Sizes stories to ~3 days, applies team-area prefixes, and writes issue keys back to the source files.

## Arguments

```
/jira-tickets --project <name>
```

- `--project` — matches folder under `docs/projects/`. **Optional if `.current-project` is set.**

**Project:** `--project <name>` or `.current-project` — stop if neither.

**Precondition:** Check `docs/projects/<name>/` for approved func-req files:

| File found & `status: approved` | Action |
|---|---|
| EA only | Run EA; note *"No approved GA functional requirements found — skipping GA."* |
| EA + GA | Run EA, then GA sequentially |
| Neither | Stop — *"No approved functional-requirements files found. Run `/func-req` first."* |

---

## Steps

### Step 1: Read functional requirements

Read `docs/projects/<name>/functional-requirements-<milestone>.md` and `docs/projects/<name>/prd.md` in full. The PRD is needed for the human-readable feature name for the Epic summary.

If the file has `status: done`, stop:
> "Tickets already created for this milestone (status: done). To add new stories, add them to the functional requirements file without JIRA keys and re-run — existing stories will be skipped."

Note all technical task groups per story and any tasks flagged `⚠️ EA blocker`.

### Step 2: Confirm JIRA project, team prefix, and label

Ask:
> "Three quick things before I create tickets:
> 1. JIRA project key? (e.g., `PROJ`, `ALERT`)
> 2. Team/area base prefix? (e.g., `WE`, `ALERT`) — I'll append the work-type suffix (`-API`, `-UI`, `-DB`, `-OPS`) per story
> 3. Project label to apply to all stories? (e.g., `Event_Enrichment`, `Q2_2026`)"

### Step 3: Create the Epic

Use JIRA MCP `createJiraIssue` with:
- **Issue type:** Epic
- **Summary:** `<EA|GA>: <Human-readable feature name>` — use the full feature name, not the slug
- **Description:** use the Epic template from [TEMPLATES.md](TEMPLATES.md)

Record the returned Epic key (e.g., `PROJ-42`).

### Step 4: Create Stories — one per technical task, sized to ~3 days

For each technical task group, create **one JIRA Story per task**. Apply sizing judgment:

- **Target: ~3 days of work per story.**
- **If a task is clearly larger than 3 days**: split into two stories before creating.
- **If multiple tasks are trivially small (<1 day)** and closely related: combine into one story.

**Summary format:** `[<TEAM-AREA>] <Feature>: <Specific Task>`

Work-type suffixes and description template: see [TEMPLATES.md](TEMPLATES.md).

For any task flagged `⚠️ EA blocker`: add a note in Implementation Details — *"Blocked on `<team/dependency>` — link JIRA dependency manually once their ticket exists."*

**Epic Link:** set to the Epic key from Step 3

**Labels:** the project label from Step 2

### Step 5: Write keys back to file

Update `docs/projects/<name>/functional-requirements-<milestone>.md`:
- Set `jira-epic-key: <EPIC-KEY>` in frontmatter
- Add JIRA issue key next to each technical task:
  ```
  API:
  - Add GET /audit-logs endpoint [PROJ-43]

  UI:
  - Date range filter component [PROJ-45]
  ```
Set `status: done` on the file.

### Step 6: Confirm completion

> "Created Epic <EPIC-KEY> and <N> stories in JIRA project <PROJECT>.
> Issue keys written back to `docs/projects/<name>/functional-requirements-<milestone>.md`.
>
> **Story count: <N> stories @ ~3 days each = ~<N×3> engineer-days estimated.**"

---

## Notes

- JIRA MCP must be configured and authenticated before running
- One story per technical task, sized to ~3 days — split larger tasks, combine trivially small ones
- Story descriptions are technical and specific — include file paths, component names, API contracts
- If a story already has a JIRA key written back to the file, skip it on re-run
- If a JIRA MCP call fails mid-run, stop and report partial state — do not retry automatically
