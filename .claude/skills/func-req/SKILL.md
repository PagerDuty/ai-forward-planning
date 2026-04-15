---
name: func-req
description: Use when a PRD is ready and you need to generate milestone-scoped functional requirements as one epic with stories for either EA or GA
---

# Functional Requirements Skill

Generate a milestone-scoped functional requirements doc (one epic + stories) from an approved PRD.

## Arguments

/func-req --project <name> --milestone ea|ga

- `--project` — matches folder under `docs/projects/`
- `--milestone` — `ea` for Early Access, `ga` for General Availability

## Steps

1. **Read the PRD**

   Read `docs/projects/<name>/prd.md`. If `status: draft`, warn:
   > "PRD status is `draft`. It hasn't been reviewed yet. Proceed anyway? (yes/no)"

2. **Read tech design (if exists)**

   Check for `docs/projects/<name>/tech-design.md`. If present, read it to inform acceptance criteria with technical detail.

3. **Extract milestone requirements**

   From the PRD, extract only the section matching `--milestone`:
   - `ea` → EA Requirements + EA Misc/Launch Tasks
   - `ga` → GA Requirements + GA Misc/Launch Tasks

   Do not mix EA and GA requirements in one doc.

4. **Define the epic**

   One epic per milestone. Title format: `[EA] <Project Name>` or `[GA] <Project Name>`.
   Epic goal = what completing this milestone achieves for users.

5. **Generate stories**

   One story per distinct user-facing capability. Rules:
   - Use the format: "As a <persona>, I want <capability> so that <outcome>."
   - Each story must have 2–5 acceptance criteria
   - Acceptance criteria must be testable (observable behavior, not implementation)
   - No tasks — only stories
   - Include Misc/Launch Tasks as stories (e.g., "As a support engineer, I want a runbook so that I can handle on-call incidents")

6. **Write file**

   Use `epic-template.md` structure. Write to:
   `docs/projects/<name>/functional-requirements-<milestone>.md`

   Frontmatter:
   project: <name>
   milestone: <ea|ga>
   created: <today's date YYYY-MM-DD>
   status: draft
   jira-epic-key:

7. **Prompt human review**

   > "Functional requirements drafted and saved to `docs/projects/<name>/functional-requirements-<milestone>.md`. Please review the stories and acceptance criteria. When ready, update `status: reviewed` and let me know."

## Notes

- Run this skill twice — once for EA, once for GA
- GA stories should be additive over EA, not repetitive
- If the PRD has a tech design, acceptance criteria should reference specific API endpoints, schema fields, or component names where relevant
