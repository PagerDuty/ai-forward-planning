# PRD → JIRA Pipeline

A set of Claude Code skills that take a PM's written PRD from concept to JIRA tickets — replacing multi-day, meeting-heavy planning with a structured, mostly-automated pipeline. Human review checkpoints exist at every decision point; artifact creation is Claude's job.

## Pipeline — 5 Phases

### Phase 1 — PM Authoring

```
PM has rough idea (no doc)     PM has written doc
         │                              │
         ▼                              ▼
/prd --project <name>     /prd --ingest --project <name> --file <path>
         │                              │
         └──────────────┬───────────────┘
                        │
                        │  PM sets status: approved
```

### Phase 2 — Scope + AI Analysis

```
/prd --enrich   (optional) Codebase scan → fill sections 5-6 + feasibility flags
      │
      │  (requires prd.md status: approved)
      ▼
/scope-propose   Propose EA/GA scope → append Scope Proposal + Meeting Brief to prd.md
                 (if sections 5-6 empty, offers to run codebase scan inline)
      │
      ├─ Needs Discussion N=0: no meeting needed → set status: scope-finalized directly
      │
      └─ Needs Discussion N>0: PM + Eng Lead joint meeting (~20 min)
              Resolve ⚡ items → move to EA or GA in prd.md
              Eng Lead sets prd.md status: scope-finalized
      │
      ▼
/tech-design   Full architecture spec on finalized scope  [draft → approved]
               (optional for UI-only features — /scope-propose will flag)
```

### Phase 3 — Functional Requirements (sequential)

```
/func-req   Generates EA first, then GA (reads EA for consistency check)
            GA auto-skipped if no GA stories in Scope Proposal
      │
      │  Requires prd.md: status: scope-finalized + tech-design.md: status: approved
      ▼
Eng Lead reviews both files in one pass
      │  status: approved
```

### Phase 4 — JIRA Ticket Creation

```
/jira-tickets   Creates Epic + Stories for EA, then GA sequentially
                (auto-sets status: done on func-req files)
      ▼
Eng Lead opens board → starts coding
```

**Total human time: ~35–45 min across 3 async reviews + 0–1 joint meetings (meeting skipped when scope has no ambiguous items). Previously: 4 async reviews + 2 meetings.**

## Skills

| Skill | Command | Purpose | Who runs it |
|-------|---------|---------|-------------|
| PRD Interview | `/prd --project <name>` | Interview PM to draft PRD from scratch | PM |
| PRD Ingest | `/prd --ingest --project <name> --file <path>` | Restructure PM's existing doc into standard format | PM |
| PRD Enrich | `/prd --enrich --project <name>` | Codebase scan → fill sections 5-6, outputs next steps | Eng Lead |
| Scope Propose | `/scope-propose --project <name>` | Propose EA/GA scope → Scope Proposal + Meeting Brief appended to prd.md; offers inline codebase scan if sections 5-6 empty | Eng Lead |
| Tech Design | `/tech-design --project <name>` | Full architecture spec on finalized scope (API, DB, infra) | Eng Lead (after status: scope-finalized) |
| Func Req | `/func-req --project <name>` | Generate EA + GA epics + stories sequentially; use `--milestone ea\|ga` for recovery only | Eng Lead |
| JIRA Tickets | `/jira-tickets --project <name>` | Create Epic + Stories in JIRA for EA then GA; use `--milestone ea\|ga` for recovery only | Eng Lead |

## Quick Start

> **Project switching:** The first command you run with `--project <name>` saves that name to `.current-project`. All subsequent commands in the same project can omit `--project`. To switch projects, run any command with a new `--project <name>`.

1. PM describes the feature — run `/prd --project <name>` (interview) or `/prd --ingest --project <name> --file <path>`
2. PM reviews the PRD and sets `status: approved` in prd.md
3. *(Optional)* Eng Lead runs `/prd --enrich --project <name>` — fills sections 5-6 with codebase context. If skipped, `/scope-propose` will offer to run the scan inline.
4. Eng Lead runs `/scope-propose --project <name>` — appends Scope Proposal and Meeting Brief to prd.md. If Needs Discussion N=0: set `status: scope-finalized` directly after PM sign-off (no meeting needed). If N>0: hold a joint meeting (~20 min) to resolve ⚡ items, then set `status: scope-finalized`.
5. Eng Lead runs `/tech-design --project <name>` — full architecture spec on the now-locked scope. *(Skip for UI-only features — `/scope-propose` will flag when no backend surfaces detected.)*
6. Eng Lead sets `tech-design.md` to `status: approved`
7. Run `/func-req --project <name>` — generates EA then GA sequentially (GA auto-skipped if no GA stories exist)
8. Eng Lead reviews both functional requirements files in one pass, sets `status: approved` on each
9. Run `/jira-tickets --project <name>` — creates Epic + Stories for EA then GA
10. Eng Lead opens the board → starts coding

## Status Vocabulary

All files use a single `status` field. Values differ by file type:

**`prd.md`** — three values:

| Value | Set by | Means |
|-------|--------|-------|
| `draft` | AI (automatic) | PRD written, not yet PM-approved |
| `approved` | PM (manual) | PM signed off — enrichment and scope can proceed |
| `scope-finalized` | Eng Lead (manual, post-meeting) | Scope locked — unlocks `/tech-design` and `/func-req` |

**All other files** — three values:

| Value | Set by | Means |
|-------|--------|-------|
| `draft` | AI (automatic) | Generated. Not yet reviewed. |
| `approved` | Human (manual) | Reviewed. Downstream skills can proceed. |
| `done` | AI (automatic) | Terminal. JIRA tickets created. |

## PRD Format

PM writes sections 1-4. Sections 5-6 are filled by `/prd --enrich` or inline during `/scope-propose` (if enrich was skipped). Scope Proposal section is appended by `/scope-propose`.

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

**Do NOT pre-split stories into EA/GA.** That is `/scope-propose`'s job.

## Project Structure

```
docs/
  projects/
    <project-name>/
      prd.md                          # /prd output — includes Scope Proposal section
      tech-design.md                  # /tech-design output
      functional-requirements-ea.md   # /func-req --milestone ea output
      functional-requirements-ga.md   # /func-req --milestone ga output
      .jira-config.md                 # saved JIRA project key, prefix, label
```

> `scope-proposal.md` no longer exists as a separate file — the Scope Proposal section lives inside `prd.md`.

## EA vs GA

| Milestone | Definition |
|-----------|-----------|
| **EA (Early Access)** | Core functionality, P0 stories, limited rollout. Known gaps documented but deferred. |
| **GA (General Availability)** | Feature-complete, all edge cases covered, production-ready. |

The EA/GA split is proposed by `/scope-propose` based on priority tags, complexity signals from the codebase scan, and dependency flags — not pre-decided by the PM.

## Prerequisites

| Requirement | Used by |
|-------------|---------|
| [Claude Code](https://claude.ai/code) with skills support | All skills |
| JIRA MCP configured and connected | `/jira-tickets` |
| GitHub MCP configured and accessible | `/prd --enrich` (offers manual fallback if unavailable), `/scope-propose` (offers inline scan; manual fallback if MCP unavailable), `/tech-design` (required — no fallback) |
