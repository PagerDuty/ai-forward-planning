---
name: jira-tickets
description: Use when functional requirements are ready and reviewed, and you need to create a JIRA Epic with Stories using JIRA MCP
---

# JIRA Tickets Skill

Create a JIRA Epic and Stories from a functional requirements doc using JIRA MCP.

## Arguments

/jira-tickets --project <name> --milestone ea|ga

- `--project` — matches folder under `docs/projects/`
- `--milestone` — `ea` or `ga`

## Steps

1. **Read functional requirements**

   Read `docs/projects/<name>/functional-requirements-<milestone>.md`.

   If `status: draft`, warn:
   > "Functional requirements status is `draft`. They haven't been reviewed yet. Proceed anyway? (yes/no)"

2. **Confirm JIRA project**

   Ask the human:
   > "Which JIRA project key should I use? (e.g., `ALERT`, `ONBOARD`)"

3. **Create the Epic**

   Use JIRA MCP to create the Epic:
   - Summary: Epic title from the requirements doc
   - Description: Epic goal from the requirements doc
   - Labels: `ea` or `ga` (matching milestone)

   Record the returned Epic key (e.g., `ALERT-42`).

4. **Create Stories**

   For each story in the requirements doc, use JIRA MCP to create a Story:
   - Summary: Story title
   - Description: Full "As a... I want... so that..." statement + Acceptance Criteria as a checklist
   - Epic Link: set to the Epic key from Step 3
   - Labels: `ea` or `ga`

5. **Write keys back to file**

   Update `docs/projects/<name>/functional-requirements-<milestone>.md`:
   - Set `jira-epic-key: <EPIC-KEY>` in frontmatter
   - Add JIRA issue key next to each story heading, e.g.:
     `### Story 1: Schedule Management [ALERT-43]`

6. **Confirm completion**

   > "Created Epic <EPIC-KEY> and <N> stories in JIRA project <PROJECT>. Issue keys written back to `docs/projects/<name>/functional-requirements-<milestone>.md`."

## Notes

- Run this skill once per milestone (ea, then ga)
- JIRA MCP must be configured and authenticated before running
- If a story already has a JIRA key in the file, skip it (idempotent)
- Do not create sub-tasks — only Stories under the Epic
- If a story has no Acceptance Criteria, omit the checklist from its JIRA description
- If a JIRA MCP call fails mid-run, stop and report the partial state (e.g., "Epic ALERT-42 created, 3 of 7 stories created"). Do not retry automatically — confirm the Epic key with the human before resuming
