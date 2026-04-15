# PRD → JIRA Pipeline — Design Spec

**Date:** 2026-04-15  
**Status:** approved  
**Author:** nwadhwa

---

## Overview

A pipeline of four Claude Code skills that takes a rough product idea from a human, collaboratively drafts a PRD, optionally produces a deep technical design, generates milestone-scoped functional requirements, and creates JIRA tickets — all from within Claude Code.

Skills are team-shareable (committed to git under `.claude/skills/`), modular (each runs independently), and file-based (plain markdown files at every stage, human-editable).

---

## 1. Scope

### In scope
- Four Claude Code skills: `/prd`, `/tech-design`, `/func-req`, `/jira-tickets`
- EA / GA milestone model embedded in PRD and respected downstream
- GitHub MCP integration for repo scanning in tech design
- JIRA MCP integration for ticket creation
- Project-namespaced output files (`docs/projects/<name>/`)
- README for orientation and quick-start

### Out of scope
- Multi-user collaboration or locking
- Automated pipeline orchestration (skills are invoked manually in sequence)
- JIRA project/board creation (assumes existing JIRA project)

---

## 2. EA / GA Milestone Model

Every PRD has two milestones:

| Milestone | Definition |
|-----------|-----------|
| **EA (Early Access)** | Core functionality, must-haves, limited users. Known gaps are documented but deferred. |
| **GA (General Availability)** | Feature-complete, all edge cases covered, production-ready. |

Both milestones include a **Misc / Launch Tasks** bucket for launch-specific work (docs, comms, rollout plan, support runbook, etc.).

---

## 3. Directory Structure

```
<repo-root>/
├── README.md
├── .claude/
│   └── skills/
│       ├── prd/
│       │   ├── SKILL.md
│       │   └── template.md            # PRD markdown template
│       ├── tech-design/
│       │   ├── SKILL.md
│       │   └── architecture-checklist.md  # GitHub MCP scan checklist
│       ├── func-req/
│       │   ├── SKILL.md
│       │   └── epic-template.md       # Epic + story structure
│       └── jira-tickets/
│           └── SKILL.md
└── docs/
    └── projects/
        └── <project-name>/
            ├── prd.md
            ├── tech-design.md         (optional)
            ├── functional-requirements-ea.md
            └── functional-requirements-ga.md
```

---

## 4. Skills

### 4.1 `/prd --project <name>`

**Purpose:** Draft a PRD from a rough idea, write it to file, prompt human review.

**Steps:**
1. Accept rough idea from human (inline or prompted)
2. Draft PRD using `template.md` structure
3. Write to `docs/projects/<name>/prd.md` with frontmatter `status: draft`
4. Instruct human: review and edit the file, then update `status: reviewed` to continue

**PRD Template Sections:**
- Problem Statement
- Goals
- Non-Goals
- EA Requirements (must-haves, limited rollout)
- GA Requirements (feature-complete, edge cases)
- Misc / Launch Tasks (EA launch tasks + GA launch tasks)
- Success Metrics

---

### 4.2 `/tech-design --project <name>` *(optional)*

**Purpose:** Generate a deep technical spec using the PRD and GitHub repo scanning.

**Steps:**
1. Read `docs/projects/<name>/prd.md` — warn if status is `draft`
2. Use **GitHub MCP** to scan referenced repos: file tree, key source files, existing architecture patterns, dependencies
3. Use `architecture-checklist.md` to guide coverage
4. Draft technical spec covering:
   - System components and responsibilities
   - Data flow and sequence diagrams
   - API contracts
   - DB schema
   - Infra / deployment notes
   - Integration points
5. Write to `docs/projects/<name>/tech-design.md` with `status: draft`
6. Prompt human review

---

### 4.3 `/func-req --project <name> --milestone ea|ga`

**Purpose:** Generate a milestone-scoped functional requirements doc as one epic with stories.

**Steps:**
1. Read `docs/projects/<name>/prd.md` — warn if status is `draft`
2. Read `docs/projects/<name>/tech-design.md` if it exists
3. Extract requirements scoped to the specified milestone (EA or GA)
4. Generate structure: **1 Epic → N Stories** (no tasks)
5. Write to `docs/projects/<name>/functional-requirements-[ea|ga].md`

**Story format:**
```
As a <persona>, I want <capability> so that <outcome>.

Acceptance Criteria:
- [ ] ...
- [ ] ...
```

---

### 4.4 `/jira-tickets --project <name> --milestone ea|ga`

**Purpose:** Create JIRA tickets from functional requirements using JIRA MCP.

**Steps:**
1. Read `docs/projects/<name>/functional-requirements-[ea|ga].md`
2. Use **JIRA MCP** to create:
   - One Epic (from epic metadata in the file)
   - One Story per story entry, linked under the Epic
3. Write JIRA issue keys back into the functional requirements file as links

---

## 5. Data Flow

```
Human rough idea
       │
       ▼
/prd --project <name>
  Writes → docs/projects/<name>/prd.md  [status: draft → reviewed → approved]
       │
       ▼  (human reviews/edits)
       │
       ├──────────────────────────────────────────┐
       │                                          │ (optional)
       ▼                                          ▼
/func-req --project <name>           /tech-design --project <name>
  --milestone ea|ga                    Reads ← prd.md
  Reads ← prd.md                       Uses GitHub MCP
  Reads ← tech-design.md (if exists)   Writes → tech-design.md
  Writes → functional-requirements-    (human reviews/edits)
           [ea|ga].md
       │
       ▼
/jira-tickets --project <name>
  --milestone ea|ga
  Reads ← functional-requirements-[ea|ga].md
  Uses JIRA MCP → creates Epic + Stories
  Writes JIRA keys back into functional-requirements file
```

---

## 6. Frontmatter Conventions

Every generated file includes YAML frontmatter:

```yaml
---
project: <name>
milestone: ea | ga | both
created: YYYY-MM-DD
status: draft | reviewed | approved
---
```

**Status gates:**
- `draft` → upstream file unreviewed; downstream skill warns before continuing
- `reviewed` → human has read it; downstream skill proceeds
- `approved` → explicitly signed off; safe to pass to JIRA

---

## 7. Skill Authoring Conventions

All skills follow official Claude Code best practices:

- `SKILL.md` has YAML frontmatter with `name` and `description` (max 1024 chars total)
- `description` starts with "Use when..." — triggering conditions only, no workflow summary
- Heavy reference material (templates, checklists) in separate files, not inline
- Skills are concise — only include context Claude doesn't already have
- Token efficiency: aim for SKILL.md < 300 words; templates/references in supporting files

---

## 8. README

`README.md` at project root covers:
- Pipeline overview with flow diagram
- Skills reference table (command, purpose, arguments)
- Quick start (rough idea → first JIRA ticket in N steps)
- Annotated project structure
- EA vs GA explanation
- Prerequisites (JIRA MCP, GitHub MCP, Claude Code)

---

## 9. Prerequisites

| Requirement | Notes |
|-------------|-------|
| Claude Code with skills support | Skills in `.claude/skills/` |
| JIRA MCP configured | Existing JIRA project required |
| GitHub MCP configured | Required only for `/tech-design` |
