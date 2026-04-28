# PRD → JIRA Pipeline

A set of Claude Code skills that take a PM's feature idea to JIRA tickets — replacing multi-day, meeting-heavy planning with a structured, mostly-automated pipeline. Human review checkpoints exist at every decision point; artifact creation is Claude's job.

## Pipeline — 4 Phases

### Phase 1 — PM Authoring

```
PM drops source doc in docs/projects/<name>/    or    no doc yet
                        │                                   │
                        └──────────────┬────────────────────┘
                                       ▼
                          /prd --project <name>
                          Auto-discovers source doc. Interviews only
                          for missing/weak sections. Challenges draft
                          (skip with --approved).
                                       │
                                       │  PM replies "approved"
                                       │  → Claude sets status: approved
```

### Phase 2 — Codebase Analysis + Scope

```
/analyze   Codebase scan → fill sections 5-6 + propose EA/GA scope
           Appends Scope Proposal to prd.md
      │
      ├─ Needs Discussion N=0: reply "approved" → proceed to /tech-design
      │
      └─ Needs Discussion N>0: hold joint meeting (~20 min)
              Resolve ⚡ items → reply "approved" → proceed to /tech-design
      │
      ▼
/tech-design   Full architecture spec on finalized scope
               (optional for UI-only features — /analyze will flag)
                                       │
                                       │  Eng Lead replies "approved"
                                       │  → Claude sets status: approved
```

### Phase 3 — Functional Requirements

```
/func-req   Generates EA first, then GA
            GA auto-skipped if no GA stories in Scope Proposal
                                       │
                                       │  Eng Lead replies "approved"
                                       │  → Claude sets status: approved on each file
```

### Phase 4 — JIRA Ticket Creation

```
/jira-tickets   Creates Epic + Stories for EA, then GA sequentially
                (auto-sets status: done on func-req files)
      ▼
Eng Lead opens board → starts coding
```

**Total human time: ~30–40 min across 2 async reviews + 0–1 joint meetings.**

## Skills

| Skill | Command | Purpose | Who runs it |
|-------|---------|---------|-------------|
| PRD | `/prd --project <name>` | Auto-discovers source doc in project folder. Interviews only for missing sections. Challenges draft (skip with `--approved`). | PM |
| Analyze | `/analyze --project <name>` | Codebase scan → fill sections 5-6 + EA/GA scope proposal | Eng Lead (after status: approved) |
| Tech Design | `/tech-design --project <name>` | Full architecture spec on finalized scope (API, DB, infra) | Eng Lead (after scope resolved) |
| Func Req | `/func-req --project <name>` | Generate EA + GA epics + stories sequentially | Eng Lead |
| JIRA Tickets | `/jira-tickets --project <name>` | Create Epic + Stories in JIRA for EA then GA | Eng Lead |

## Quick Start

> **Project switching:** The first command you run with `--project <name>` saves that name to `.current-project`. All subsequent commands in the same project can omit `--project`. To switch projects, run any command with a new `--project <name>`.

1. PM drops their source doc (any `.md`) into `docs/projects/<name>/` and runs `/prd --project <name>`. Claude reads what's there, interviews only for gaps, and challenges the draft. Use `--approved` to skip the challenge.
2. PM replies **"approved"** → Claude sets `status: approved` in `prd.md` and prompts next step.
3. Eng Lead runs `/analyze --project <name>` — fills sections 5-6 with codebase context and appends Scope Proposal. If N=0 ⚡ items: reply **"approved"** to proceed. If N>0: resolve ⚡ items in the Scope Proposal, then reply **"approved"**.
4. Eng Lead runs `/tech-design --project <name>` — full architecture spec on locked scope. *(Skip for UI-only features — `/analyze` will flag.)*
5. Eng Lead replies **"approved"** → Claude sets `status: approved` in `tech-design.md` and prompts next step.
6. Run `/func-req --project <name>` — generates EA then GA sequentially (GA auto-skipped if no GA stories).
7. Eng Lead replies **"approved"** → Claude sets `status: approved` on each functional requirements file and prompts next step.
8. Run `/jira-tickets --project <name>` — creates Epic + Stories for EA then GA.
9. Eng Lead opens the board → starts coding.

## Status Vocabulary

One status field, same three values across all files:

| Value | Set by | Means |
|-------|--------|-------|
| `draft` | AI (automatic) | Generated. Not yet reviewed. |
| `approved` | Human (reply "approved") | Reviewed and signed off. Claude updates the file. Downstream skills can proceed. |
| `done` | AI (automatic) | Terminal. JIRA tickets created. |

`prd.md` only uses `draft` and `approved`. Scope readiness is determined by the Scope Proposal section contents (no ⚡ items = scope is resolved), not a separate status value.

## PRD Format

PM writes sections 1-4. Sections 5-6 are filled by `/analyze`. Scope Proposal section is appended by `/analyze`.

**Required sections (PM):**
- **Problem Statement** — who has the problem, what it is, why now
- **Goals** — 3-5 measurable outcomes (not features)
- **Non-Goals** — explicit exclusions (required — prevents scope creep)
- **User Stories** — `US-XX [P0/P1/P2] As a <persona>, I want <capability> so that <outcome>`

**Priority tags:**
| Tag | Meaning |
|-----|---------|
| `[P0]` | Must have — feature is useless without this |
| `[P1]` | Important — should ship soon but can be deferred |
| `[P2]` | Nice to have — cut if needed |

**Do NOT pre-split stories into EA/GA.** That is `/analyze`'s job.

## Project Structure

```
docs/
  projects/
    <project-name>/
      prd.md                          # /prd output — includes Scope Proposal section (appended by /analyze)
      tech-design.md                  # /tech-design output
      functional-requirements-ea.md   # /func-req output
      functional-requirements-ga.md   # /func-req output (if GA scope exists)
```

## EA vs GA

| Milestone | Definition |
|-----------|-----------|
| **EA (Early Access)** | Core functionality, P0 stories, limited rollout. Known gaps documented but deferred. |
| **GA (General Availability)** | Feature-complete, all edge cases covered, production-ready. |

The EA/GA split is proposed by `/analyze` based on priority tags, complexity signals from the codebase scan, and dependency flags — not pre-decided by the PM.

## Prerequisites

| Requirement | Used by |
|-------------|---------|
| [Claude Code](https://claude.ai/code) with skills support | All skills |
| JIRA MCP configured and connected | `/jira-tickets` |
| GitHub MCP configured and accessible | `/analyze` (required — fails fast if unavailable), `/tech-design` (required) |
| Figma MCP (optional) | `/prd`, `/analyze`, `/tech-design`, `/func-req` — used when `design-mocks` is set in `prd.md` |
