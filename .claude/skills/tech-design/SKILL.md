---
name: tech-design
description: "Use when a PRD is approved and scope is finalized (Scope Proposal in prd.md has no remaining ⚡ items). Covers system design, API contracts, DB schema, and infra. Requires GitHub MCP. Run after /analyze and the joint alignment meeting."
---

# Tech Design Skill

Generate a deep technical spec from an approved PRD with a finalized Scope Proposal. Uses GitHub MCP to scan existing architecture. Scoped to the EA/GA cut from the Scope Proposal section of `prd.md`.

## Arguments

```
/tech-design --project <name>
```

- `--project` — matches the folder under `docs/projects/`. **Optional if `.current-project` is set.**

**Resolve project:** Use `--project <name>` if provided (writes to `.current-project`). Else read `.current-project`. If neither, stop: *"No project set. Run with `--project <name>` or set `.current-project`."*

**Preconditions:**
1. `prd.md` must have `status: approved` — if not, stop: *"PRD not approved. Set `status: approved` in prd.md before running tech design."*
2. `prd.md` must contain a **Scope Proposal** section with EA/GA assignments resolved (no remaining ⚡ items) — if missing or unresolved, stop:
   > "Scope Proposal is missing or has unresolved ⚡ items. Run `/analyze --project <name>` and complete the alignment meeting before running tech design. Tech design on unfinalized scope produces rework."

---

## Steps

### Step 1: Read inputs

Read before doing anything else:

1. `docs/projects/<name>/prd.md` — full context: problem statement, goals, non-goals, user stories, affected surfaces (section 5), dependencies & constraints (section 6), and the Scope Proposal section. Check `design-mocks` in frontmatter.

2. **Fetch design mocks (if present):** If `design-mocks` is set and non-empty, use `mcp__claude_ai_Figma__get_design_context`. Extract: screens per story, UI components and their interactions, data displayed per screen (informs API response shape), form inputs and validation patterns (informs API request shape), and any states that imply background work (loading, polling, async operations). Use this throughout Steps 3-4 to ground API contracts and component design in what's actually designed — not inferred from text.

Use the Scope Proposal section in `prd.md` for the approved EA/GA cut. Sections 5-6 are a starting point for the codebase scan — these surfaces and dependencies were already identified by `/analyze`. The scan goes deeper, not wider.

### Step 2: Identify repos to scan

Based on the affected surfaces in section 5, ask:
> "Based on the affected surfaces, I need to scan the relevant repos. Which repos contain: [list surfaces from section 5]? Provide as `org/repo`."

If the engineering lead cannot identify a repo for a surface, note it as "Repo unknown — manual identification required" in the Open Technical Questions section and proceed with the repos that are identified.

### Step 3: Scan repos with GitHub MCP

For each repo, focus on areas relevant to the affected surfaces — don't scan everything.

Begin from the specific files already identified in PRD sections 5-6 as entry points. Scan deeper from those locations rather than from the repo root.

For each relevant surface, extract:
- File tree (top 2 levels) to understand structure
- Route/controller files for API surfaces
- Schema/migration files for DB surfaces
- Component files for UI surfaces
- Worker/job files for background processing surfaces
- Service boundary definitions and integration points

### Step 4: Draft the tech design

Write a doc covering EA and GA scope separately where the work differs. For sections identical across milestones, note that once.

---

#### System Overview
2–3 paragraph narrative of the architecture. What are the major components and how do they relate to this feature?

#### Components
Table: Component | Responsibility | Technology | Owned by | EA / GA / Both

#### Data Flow
Numbered sequence for the primary use case. For complex flows, include a sequence diagram in mermaid.

#### API Contracts
For each new or modified endpoint, tag with `[EA]` or `[GA]`:

```
[EA] GET /audit-logs
Auth: Bearer token (existing middleware)
Request params: date_from (ISO8601), date_to (ISO8601), user_id (optional)
Response 200: { logs: [{ id, user_id, action, timestamp }], total, page }
Response 400: { error: string }
Response 403: { error: "Insufficient permissions" }
```

#### Database Schema
For each new or modified table/collection, tag with `[EA]` or `[GA]`:

```sql
-- [EA] New index on existing table
CREATE INDEX idx_events_user_id ON events(user_id);

-- [GA] New table for retention policies
CREATE TABLE audit_retention_policies (
  id UUID PRIMARY KEY,
  workspace_id UUID NOT NULL REFERENCES workspaces(id),
  retention_days INT NOT NULL DEFAULT 90,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

#### Infra & Deployment
- New services, queues, or storage required (tag EA/GA)
- Environment variable changes
- Migration strategy (zero-downtime?)
- Rollback plan

#### Integration Points
- External services touched
- Webhooks sent or received
- Events published or consumed
- Cross-team dependencies (flag if these are blockers for EA)

#### Open Technical Questions
Unresolved design decisions that need an answer before implementation begins. Flag anything that blocks EA specifically.

---

### Step 5: Write file

Write to `docs/projects/<name>/tech-design.md`:

```markdown
---
project: <name>
created: <today's date YYYY-MM-DD>
status: draft
---

<!-- repos-scanned: [org/repo, ...] -->
```

### Step 6: Prompt engineering lead review

> "Tech design drafted and saved to `docs/projects/<name>/tech-design.md`.
>
> Review the draft:
> 1. Verify `[EA]` / `[GA]` tags on API contracts, DB schema, and components are correct
> 2. Resolve or flag any open technical questions
> 3. Set `status: approved` to proceed to `/func-req`"

---

## Notes

- This skill is optional — skip for pure UX/product changes with no backend or data model work. `/analyze` will flag when all surfaces are UI-only; `/func-req` will offer a UX-only path if `tech-design.md` is absent.
- Scan only the repos relevant to the affected surfaces — not the entire org
- Sections 5-6 in the enriched PRD are a starting point, not the complete picture — the deep scan may surface additional dependencies
- **Re-entry after scope change:** If scope decisions change significantly after `status: approved`, set `tech-design.md` back to `status: draft`, revise the relevant sections, and re-approve. Do not leave an approved tech design that no longer reflects the agreed scope.
