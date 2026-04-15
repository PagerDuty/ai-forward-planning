# PRD → JIRA Pipeline — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build four Claude Code skills that automate the pipeline from a rough product idea to JIRA tickets, following EA/GA milestones.

**Architecture:** File-based handoff — each skill reads/writes markdown files under `docs/projects/<name>/`. Skills live in `.claude/skills/` for team sharing. YAML frontmatter status gates (`draft` → `reviewed` → `approved`) control downstream progression.

**Tech Stack:** Claude Code skills (SKILL.md), Markdown, JIRA MCP, GitHub MCP

---

## File Map

| File | Purpose |
|------|---------|
| `README.md` | Pipeline overview, quick start, skills reference |
| `.claude/skills/prd/SKILL.md` | PRD skill instructions |
| `.claude/skills/prd/template.md` | PRD markdown template |
| `.claude/skills/tech-design/SKILL.md` | Tech design skill instructions |
| `.claude/skills/tech-design/architecture-checklist.md` | GitHub MCP scan checklist |
| `.claude/skills/func-req/SKILL.md` | Functional requirements skill instructions |
| `.claude/skills/func-req/epic-template.md` | Epic + story structure template |
| `.claude/skills/jira-tickets/SKILL.md` | JIRA tickets skill instructions |
| `docs/projects/.gitkeep` | Placeholder to track empty directory |

---

### Task 1: Project Setup

**Files:**
- Create: `.gitignore`
- Create: `docs/projects/.gitkeep`
- Create: `.claude/skills/` directory tree

- [ ] **Step 1: Initialize git repo**

```bash
cd /Users/nwadhwa/prd-to-jira
git init
```
Expected: `Initialized empty Git repository in /Users/nwadhwa/prd-to-jira/.git/`

- [ ] **Step 2: Create .gitignore**

Create `.gitignore`:
```
.DS_Store
*.swp
*.swo
```

- [ ] **Step 3: Create directory structure**

```bash
mkdir -p .claude/skills/prd
mkdir -p .claude/skills/tech-design
mkdir -p .claude/skills/func-req
mkdir -p .claude/skills/jira-tickets
mkdir -p docs/projects
touch docs/projects/.gitkeep
```

- [ ] **Step 4: Initial commit**

```bash
git add .gitignore docs/projects/.gitkeep
git commit -m "chore: initialize project structure"
```

---

### Task 2: README

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README.md**

Create `README.md`:

```markdown
# PRD → JIRA Pipeline

A set of Claude Code skills that take a rough product idea from concept to JIRA tickets — with human review checkpoints at every stage.

## Pipeline

```
Rough idea → /prd → [review] → /tech-design (optional) → /func-req --milestone ea|ga → /jira-tickets --milestone ea|ga
```

## Skills

| Skill | Command | Purpose |
|-------|---------|---------|
| PRD | `/prd --project <name>` | Draft a PRD from a rough idea |
| Tech Design | `/tech-design --project <name>` | Generate deep technical spec from PRD + GitHub repos |
| Functional Requirements | `/func-req --project <name> --milestone ea\|ga` | Generate epic + stories scoped to a milestone |
| JIRA Tickets | `/jira-tickets --project <name> --milestone ea\|ga` | Create Epic + Stories in JIRA |

## Quick Start

1. Have a rough idea for a feature
2. Run `/prd --project my-feature`
3. Review and edit `docs/projects/my-feature/prd.md`, update `status: reviewed`
4. *(Optional)* Run `/tech-design --project my-feature` if the feature needs architecture work
5. Run `/func-req --project my-feature --milestone ea` to generate EA requirements
6. Run `/jira-tickets --project my-feature --milestone ea` to create EA JIRA tickets
7. Repeat steps 5–6 with `--milestone ga` for GA

## EA vs GA

| Milestone | Definition |
|-----------|-----------|
| **EA (Early Access)** | Core functionality, must-haves, limited users. Known gaps documented but deferred. |
| **GA (General Availability)** | Feature-complete, all edge cases covered, production-ready. |

Both milestones include a **Misc / Launch Tasks** bucket for rollout-specific work.

## Project Structure

```
docs/
  projects/
    <project-name>/
      prd.md                          # /prd output
      tech-design.md                  # /tech-design output (optional)
      functional-requirements-ea.md   # /func-req --milestone ea output
      functional-requirements-ga.md   # /func-req --milestone ga output
```

## Prerequisites

- [Claude Code](https://claude.ai/code) with skills support
- JIRA MCP configured and connected
- GitHub MCP configured (required only for `/tech-design`)
```

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add README with pipeline overview and quick start"
```

---

### Task 3: PRD Skill — Template

**Files:**
- Create: `.claude/skills/prd/template.md`

- [ ] **Step 1: Write template.md**

Create `.claude/skills/prd/template.md`:

```markdown
---
project: {{project-name}}
milestone: both
created: {{date}}
status: draft
---

# {{Project Name}} — PRD

## Problem Statement
<!--
What problem are we solving?
Who has this problem? (persona, scale, frequency)
What happens today without this solution?
-->

## Goals
<!--
What outcomes do we want to achieve?
Keep to 3–5 measurable goals.
-->
-

## Non-Goals
<!--
What are we explicitly NOT doing in this scope?
This prevents scope creep and sets stakeholder expectations.
-->
-

---

## EA Requirements (Early Access)
> Core functionality, must-haves, limited rollout. Known gaps are documented but deferred to GA.

### Must-Haves
<!--
List requirements as user-facing capabilities, not implementation details.
Each item should be testable.
-->
-

### Known Gaps / Deferred to GA
<!--
What are we knowingly leaving out for EA?
-->
-

### Misc / Launch Tasks (EA)
<!--
Launch-specific work: docs, comms, onboarding, rollout plan, support runbook.
-->
- [ ]

---

## GA Requirements (General Availability)
> Feature-complete, all edge cases handled, production-ready.

### Features
<!--
Everything from EA, plus what gets added for GA.
-->
-

### Edge Cases
<!--
Corner cases, error states, load/scale considerations.
-->
-

### Misc / Launch Tasks (GA)
<!--
GA-specific launch tasks: SLAs, full rollout, deprecation of workarounds, etc.
-->
- [ ]

---

## Success Metrics
<!--
How do we know this is working?
Include leading indicators (adoption) and lagging indicators (outcome).
-->
-

---

## Open Questions
<!--
Unresolved decisions that need answers before or during development.
-->
-
```

- [ ] **Step 2: Commit**

```bash
git add .claude/skills/prd/template.md
git commit -m "feat: add PRD template"
```

---

### Task 4: PRD Skill — SKILL.md

**Files:**
- Create: `.claude/skills/prd/SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Create `.claude/skills/prd/SKILL.md`:

```markdown
---
name: prd
description: Use when starting a new product feature or initiative and you have a rough idea to turn into a structured PRD with EA and GA milestones
---

# PRD Skill

Draft a structured PRD from a rough idea, save it for human review, and prepare it for downstream pipeline skills.

## Arguments

```
/prd --project <name>
```

- `--project` — short slug for the product/feature (e.g., `alerting-v2`, `onboarding-flow`). Used as the folder name under `docs/projects/`.

## Steps

1. **Collect the idea** — Ask the human for their rough idea if not provided inline. One open question: "What problem are you trying to solve and who has it?"

2. **Draft the PRD** — Fill in all sections of `template.md`. Be opinionated:
   - Separate EA (must-haves, limited users) from GA (feature-complete, all edge cases)
   - EA Known Gaps should list anything you're intentionally deferring
   - Both milestones need a Misc / Launch Tasks section
   - Success Metrics must be measurable, not vague

3. **Create project directory and write file**

   ```bash
   mkdir -p docs/projects/<name>
   ```

   Write the drafted PRD to `docs/projects/<name>/prd.md` with frontmatter:
   ```yaml
   ---
   project: <name>
   milestone: both
   created: <today's date YYYY-MM-DD>
   status: draft
   ---
   ```

4. **Prompt human review** — Say exactly:
   > "PRD drafted and saved to `docs/projects/<name>/prd.md`. Please review and edit it. When you're happy with it, update `status: reviewed` in the frontmatter and let me know."

## Status Gates

Downstream skills check the `status` field:
- `draft` — not yet reviewed by human
- `reviewed` — human has read and edited it
- `approved` — explicitly signed off, safe to pass to JIRA

## Notes

- Follow the template structure exactly — downstream skills parse section headings
- EA and GA sections are both required even if GA is minimal
- Do not invent requirements — only capture what the human described
```

- [ ] **Step 2: Commit**

```bash
git add .claude/skills/prd/SKILL.md
git commit -m "feat: add /prd skill"
```

- [ ] **Step 3: Validate the skill**

In a new Claude Code session, run:
```
/prd --project test-validation
```
Provide rough idea: "I want users to be able to set up on-call schedules and get paged when alerts fire."

Verify:
- [ ] Skill prompts for or accepts rough idea
- [ ] Drafts PRD with all sections (Problem, Goals, Non-Goals, EA, GA, Metrics)
- [ ] EA and GA sections are distinct and populated
- [ ] File written to `docs/projects/test-validation/prd.md`
- [ ] Frontmatter has `status: draft`
- [ ] Human review prompt is shown

Clean up: `rm -rf docs/projects/test-validation`

---

### Task 5: Tech Design Skill — Architecture Checklist

**Files:**
- Create: `.claude/skills/tech-design/architecture-checklist.md`

- [ ] **Step 1: Write architecture-checklist.md**

Create `.claude/skills/tech-design/architecture-checklist.md`:

```markdown
# Architecture Scan Checklist

Use this checklist when scanning GitHub repos with GitHub MCP to inform the tech design doc.

## 1. Repo Structure

- [ ] Top-level directory layout (what lives where?)
- [ ] Key entry points: `main.*`, `index.*`, `app.*`, `server.*`
- [ ] Config files: `.env.example`, `config/`, `settings.*`
- [ ] Infrastructure: `Dockerfile`, `docker-compose.yml`, `k8s/`, `terraform/`, `.github/workflows/`

## 2. Tech Stack

- [ ] Language(s) and runtime versions (`package.json`, `go.mod`, `requirements.txt`, `Gemfile`, `pom.xml`)
- [ ] Frameworks (web framework, ORM, test framework)
- [ ] Key dependencies (message queues, caches, search engines)
- [ ] Database type and migration tooling

## 3. Data Layer

- [ ] Database schema files (`migrations/`, `schema.sql`, `models/`)
- [ ] ORM models or data classes
- [ ] Caching strategy (Redis keys, TTLs)
- [ ] Data retention or archival patterns

## 4. API Surface

- [ ] API definition files (`openapi.yaml`, `swagger.json`, `*.proto`, `schema.graphql`)
- [ ] Router files (list of endpoints)
- [ ] Auth middleware (how is authentication handled?)
- [ ] Rate limiting or quota patterns

## 5. Service Boundaries

- [ ] Service-to-service communication (REST, gRPC, events, queues)
- [ ] Event/message topics or queues (Kafka topics, SQS queues, PubSub)
- [ ] External integrations (third-party APIs, webhooks)

## 6. Observability

- [ ] Logging patterns (structured logs, log levels)
- [ ] Metrics instrumentation (Prometheus, StatsD, Datadog)
- [ ] Tracing setup (OpenTelemetry, Jaeger)
- [ ] Alerting config files

## 7. Deployment

- [ ] CI/CD pipeline files
- [ ] Environment promotion strategy (dev → staging → prod)
- [ ] Feature flag system (if any)
- [ ] Secrets management approach

## 8. Test Patterns

- [ ] Test directory structure
- [ ] Unit vs integration vs e2e split
- [ ] Test fixtures or factories
- [ ] Coverage tooling

---

## Output Format for Each Repo

After scanning, summarize findings as:

```
### Repo: <org/repo>

**Stack:** <language>, <framework>, <DB>
**Entry point:** <file>
**Key patterns:** <2-3 bullet observations>
**Relevant to this PRD:** <how it informs the tech design>
```
```

- [ ] **Step 2: Commit**

```bash
git add .claude/skills/tech-design/architecture-checklist.md
git commit -m "feat: add architecture scan checklist for tech-design skill"
```

---

### Task 6: Tech Design Skill — SKILL.md

**Files:**
- Create: `.claude/skills/tech-design/SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Create `.claude/skills/tech-design/SKILL.md`:

```markdown
---
name: tech-design
description: Use when a PRD exists and the feature requires a deep technical architecture spec — covers system design, API contracts, DB schema, and infra. Requires GitHub MCP.
---

# Tech Design Skill

Generate a deep technical spec from an approved PRD, using GitHub MCP to scan existing architecture.

## Arguments

```
/tech-design --project <name>
```

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
   ```
   METHOD /path
   Auth: <mechanism>
   Request: { field: type, ... }
   Response 200: { field: type, ... }
   Response 4xx/5xx: { error: string }
   ```

   ### Database Schema
   For each new or modified table/collection:
   ```sql
   CREATE TABLE name (
     id UUID PRIMARY KEY,
     ...
     created_at TIMESTAMPTZ NOT NULL DEFAULT now()
   );
   ```

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
   ```yaml
   ---
   project: <name>
   created: <today's date YYYY-MM-DD>
   repos-scanned: [org/repo, ...]
   status: draft
   ---
   ```

6. **Prompt human review**

   > "Tech design drafted and saved to `docs/projects/<name>/tech-design.md`. Please review it. When ready, update `status: reviewed` and let me know."

## Notes

- This skill is optional — not every PRD needs a tech design doc
- If the PRD is purely a UX/product change with no backend work, skip this skill
- Scan only the repos relevant to the PRD — don't scan everything
```

- [ ] **Step 2: Commit**

```bash
git add .claude/skills/tech-design/SKILL.md
git commit -m "feat: add /tech-design skill"
```

---

### Task 7: Functional Requirements Skill — Epic Template

**Files:**
- Create: `.claude/skills/func-req/epic-template.md`

- [ ] **Step 1: Write epic-template.md**

Create `.claude/skills/func-req/epic-template.md`:

```markdown
---
project: {{project-name}}
milestone: {{ea|ga}}
created: {{date}}
status: draft
jira-epic-key:
---

# Epic: {{Epic Title}}

**Milestone:** {{EA | GA}}
**Goal:** {{One sentence: what does completing this epic achieve?}}
**Source:** `docs/projects/{{project-name}}/prd.md`

---

## Stories

### Story 1: {{Story Title}}

**As a** {{persona}}, **I want** {{capability}} **so that** {{outcome}}.

**Acceptance Criteria:**
- [ ] {{Criterion 1}}
- [ ] {{Criterion 2}}
- [ ] {{Criterion 3}}

**Notes:** {{Any implementation notes, edge cases, or dependencies}}

---

### Story 2: {{Story Title}}

**As a** {{persona}}, **I want** {{capability}} **so that** {{outcome}}.

**Acceptance Criteria:**
- [ ] {{Criterion 1}}
- [ ] {{Criterion 2}}

**Notes:** {{Any implementation notes, edge cases, or dependencies}}

---

<!-- Repeat for each story. No tasks — only stories under the epic. -->
```

- [ ] **Step 2: Commit**

```bash
git add .claude/skills/func-req/epic-template.md
git commit -m "feat: add epic + story template for func-req skill"
```

---

### Task 8: Functional Requirements Skill — SKILL.md

**Files:**
- Create: `.claude/skills/func-req/SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Create `.claude/skills/func-req/SKILL.md`:

```markdown
---
name: func-req
description: Use when a PRD is ready and you need to generate milestone-scoped functional requirements as one epic with stories for either EA or GA
---

# Functional Requirements Skill

Generate a milestone-scoped functional requirements doc (one epic + stories) from an approved PRD.

## Arguments

```
/func-req --project <name> --milestone ea|ga
```

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
   ```yaml
   ---
   project: <name>
   milestone: <ea|ga>
   created: <today's date YYYY-MM-DD>
   status: draft
   jira-epic-key:
   ---
   ```

7. **Prompt human review**

   > "Functional requirements drafted and saved to `docs/projects/<name>/functional-requirements-<milestone>.md`. Please review the stories and acceptance criteria. When ready, update `status: reviewed` and let me know."

## Notes

- Run this skill twice — once for EA, once for GA
- GA stories should be additive over EA, not repetitive
- If the PRD has a tech design, acceptance criteria should reference specific API endpoints, schema fields, or component names where relevant
```

- [ ] **Step 2: Commit**

```bash
git add .claude/skills/func-req/SKILL.md
git commit -m "feat: add /func-req skill"
```

- [ ] **Step 3: Validate the skill**

In a new Claude Code session (with a test prd.md in place), run:
```
/func-req --project test-validation --milestone ea
```

Verify:
- [ ] Skill reads prd.md and warns if status is `draft`
- [ ] Extracts only EA requirements (not GA)
- [ ] Produces exactly one epic
- [ ] Stories follow "As a... I want... so that..." format
- [ ] Each story has acceptance criteria
- [ ] No tasks in output — only stories
- [ ] Misc/Launch Tasks are included as stories
- [ ] File written to `docs/projects/test-validation/functional-requirements-ea.md`
- [ ] Frontmatter has `jira-epic-key:` (empty, to be filled by jira-tickets skill)

---

### Task 9: JIRA Tickets Skill — SKILL.md

**Files:**
- Create: `.claude/skills/jira-tickets/SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Create `.claude/skills/jira-tickets/SKILL.md`:

```markdown
---
name: jira-tickets
description: Use when functional requirements are ready and reviewed, and you need to create a JIRA Epic with Stories using JIRA MCP
---

# JIRA Tickets Skill

Create a JIRA Epic and Stories from a functional requirements doc using JIRA MCP.

## Arguments

```
/jira-tickets --project <name> --milestone ea|ga
```

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
```

- [ ] **Step 2: Commit**

```bash
git add .claude/skills/jira-tickets/SKILL.md
git commit -m "feat: add /jira-tickets skill"
```

---

### Task 10: Final Validation & Cleanup

- [ ] **Step 1: Verify all files exist**

```bash
find .claude/skills -name "*.md" | sort
```

Expected output:
```
.claude/skills/func-req/SKILL.md
.claude/skills/func-req/epic-template.md
.claude/skills/jira-tickets/SKILL.md
.claude/skills/prd/SKILL.md
.claude/skills/prd/template.md
.claude/skills/tech-design/SKILL.md
.claude/skills/tech-design/architecture-checklist.md
```

- [ ] **Step 2: Verify git log**

```bash
git log --oneline
```

Expected: 8+ commits covering setup, README, each skill and template.

- [ ] **Step 3: Run end-to-end smoke test**

In a new Claude Code session:
1. `/prd --project smoke-test` → provide idea: "Users need to acknowledge alerts before they escalate"
2. Manually edit `docs/projects/smoke-test/prd.md`, set `status: reviewed`
3. `/func-req --project smoke-test --milestone ea`
4. Verify `docs/projects/smoke-test/functional-requirements-ea.md` exists with epic + stories

Clean up: `rm -rf docs/projects/smoke-test`

- [ ] **Step 4: Final commit**

```bash
git add -A
git commit -m "chore: complete pipeline implementation"
```
