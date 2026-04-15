---
name: tech-design
description: Use when a PRD exists and the feature requires a deep technical architecture spec — covers system design, API contracts, DB schema, and infra. Requires GitHub MCP.
---

# Tech Design Skill

Generate a deep technical spec from an approved PRD, using GitHub MCP to scan existing architecture.

## Arguments

/tech-design --project <name>

- `--project` — matches the folder under `docs/projects/`

## Steps

1. **Read the PRD**

   Read `docs/projects/<name>/prd.md`. If `status: draft`, warn:
   > "PRD status is `draft`. It hasn't been reviewed yet. Proceed anyway? (yes/no)"

2. **Identify repos to scan**

   Ask the human:
   > "Which GitHub repos should I scan? Provide one or more as `org/repo` (e.g., `acme/backend`, `acme/infra`)."

3. **Scan repos with GitHub MCP**

   For each repo, work through `architecture-checklist.md` section by section. Extract:
   - File tree (top 2 levels)
   - Contents of key files identified in the checklist
   - Summarize findings per the checklist output format

4. **Draft the tech design**

   Write a doc with these sections:

   ### System Overview
   2–3 paragraph narrative of the architecture. What are the major components and how do they relate?

   ### Components
   Table: Component | Responsibility | Technology | Owned by

   ### Data Flow
   Numbered sequence for the primary use case. For complex flows, include a sequence diagram in plain text or mermaid.

   ### API Contracts
   For each new or modified endpoint:

   METHOD /path
   Auth: <mechanism>
   Request: { field: type, ... }
   Response 200: { field: type, ... }
   Response 4xx/5xx: { error: string }

   ### Database Schema
   For each new or modified table/collection:

   CREATE TABLE name (
     id UUID PRIMARY KEY,
     ...
     created_at TIMESTAMPTZ NOT NULL DEFAULT now()
   );

   ### Infra & Deployment
   - New services, queues, or storage required
   - Environment variable changes
   - Migration strategy (zero-downtime?)
   - Rollback plan

   ### Integration Points
   - External services touched
   - Webhooks sent or received
   - Events published or consumed

   ### Open Technical Questions
   Unresolved design decisions that need an answer before implementation.

5. **Write file**

   Write to `docs/projects/<name>/tech-design.md` with frontmatter:

   project: <name>
   created: <today's date YYYY-MM-DD>
   repos-scanned: [org/repo, ...]
   status: draft

6. **Prompt human review**

   > "Tech design drafted and saved to `docs/projects/<name>/tech-design.md`. Please review it. When ready, update `status: reviewed` and let me know."

## Notes

- This skill is optional — not every PRD needs a tech design doc
- If the PRD is purely a UX/product change with no backend work, skip this skill
- Scan only the repos relevant to the PRD — don't scan everything
