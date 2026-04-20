# AI-Native PRD-to-JIRA Pipeline — Design Spec

**Date:** 2026-04-19  
**Status:** approved  
**Author:** PM + Claude (expert PM / solution architect review)

---

## Problem Statement

The current pipeline is a linear, sequential, artifact-generation workflow. It works but has three structural weaknesses in an AI-native team context:

1. **Sequential bottleneck** — scope-propose and tech-design run one after another, requiring 3–4 separate human review cycles and multiple async handoffs.
2. **Passive AI role** — AI formats and generates; it does not challenge, question, or learn. Humans still do the analytical work of spotting bad assumptions.
3. **No feedback loop** — post-delivery learnings from JIRA (stories split, tickets too vague, complexity misestimates) are never fed back into future runs. The pipeline doesn't get smarter.

Additionally, the current status vocabulary is inconsistent across files (`pm-complete`, `reviewed`, `scope-approved`, `final`, `tickets-created`), making it hard to remember and error-prone.

---

## Goals

1. Reduce human review cycles from 4 async cycles + 2 meetings to 2 async reviews + 1 joint meeting.
2. Unify status vocabulary to a single set used across all artifact files.
3. Add AI challenge and feedback loop steps as first-class pipeline phases.
4. Clarify role boundaries: AI generates and challenges, humans decide and approve.
5. Eliminate `scope-proposal.md` as a separate file — merge into `prd.md`.

---

## Non-Goals

- Removing human checkpoints — every phase retains a human approval gate.
- Fully automating ticket creation without human review.
- Replacing PM or Eng Lead judgment on scope or technical decisions.
- Changing the JIRA ticket structure (one story per technical task stays).

---

## Architecture

### Unified Status Vocabulary

Three statuses applied consistently across all artifact files:

| Status | Meaning | Set by |
|---|---|---|
| `draft` | AI-generated. Not yet human-approved. Speculative drafts also use this. | AI (automatic) |
| `approved` | Human confirmed. Downstream skills can proceed. | Human (manual) |
| `done` | Terminal state. JIRA tickets created. No further action. | AI (automatic) |

Speculative drafts (generated during parallel AI analysis before scope is finalized) use `draft` status and include a banner at the top of the file:

```
> ⚠️ Speculative draft — generated before scope was finalized. Review after the joint alignment meeting and remove this banner when approving.
```

Humans only ever need to remember one word: `approved`.

---

### Simplified File Structure

| File | Contents | Change from current |
|---|---|---|
| `prd.md` | Problem, Goals, Non-Goals, User Stories, Affected Surfaces, Dependencies, **Scope Proposal section** | Scope proposal merged in — `scope-proposal.md` eliminated |
| `tech-design.md` | Architecture, API Contracts, DB Schema, Infra, Open Questions | `repos-scanned` moved from frontmatter to a comment block inside the doc |
| `functional-requirements-ea.md` | EA epic + stories + technical tasks | Unchanged |
| `functional-requirements-ga.md` | GA epic + stories + technical tasks | Unchanged |
| `retro.md` | Post-delivery calibration findings | New |
| `.jira-config.md` | JIRA project key, prefix, label | Unchanged |

---

### Pipeline — 5 Phases

#### Phase 1 — PM Authoring

**Skills:** `/prd` (unchanged) + `/challenge` (new)

1. PM runs `/prd --project <name>` (interview) or `/prd --ingest` (existing doc).
2. AI runs `/challenge --project <name>` — reviews the drafted PRD and produces a structured challenge report: surfaced assumptions, missing personas, contradictions between stories, stories that are too vague to scope. PM reviews and responds in the same file.
3. PM sets `status: approved` on `prd.md` sections 1–4 when satisfied.

**Human checkpoint:** PM sets `approved` on `prd.md`.

---

#### Phase 2 — Parallel AI Analysis (restructured)

**Skills:** `/prd --enrich` → auto-triggers `/scope-propose` + `/tech-design` in parallel → `/pre-meeting-brief` (new)

1. Eng Lead runs `/prd --enrich --project <name>`. Fills sections 5–6 (Affected Surfaces, Dependencies & Constraints) via codebase scan.
2. On completion, AI automatically triggers two speculative drafts in parallel:
   - **Track A:** `/scope-propose` — scores stories using enrich output; proposes EA/GA cut; appended as a **Scope Proposal** section at the bottom of `prd.md` (not a separate file). Status: `draft`.
   - **Track B:** `/tech-design` — generates full-scope architecture (API, DB, infra) without an EA/GA split. EA/GA tags within the doc are left blank. Status: `draft`.
3. AI runs `/pre-meeting-brief --project <name>` — auto-generates a 1-page decision summary containing: debatable scope items (Needs Discussion), EA blockers, open technical questions, risk flags. Written to `docs/projects/<name>/pre-meeting-brief.md`.
4. PM + Eng Lead hold one joint meeting (~30 min) reviewing the brief:
   - Finalize scope cut: resolve Needs Discussion items directly in the Scope Proposal section of `prd.md`
   - Validate tech design: confirm API contracts, flag any open questions as resolved or blocked
   - After meeting: Eng Lead adds EA/GA tags to `tech-design.md` sections
5. Both `prd.md` and `tech-design.md` set to `status: approved`.

**Human checkpoint:** PM + Eng Lead joint meeting → both files set to `approved`.

**Why this works:** After `/prd --enrich`, all context needed for both skills is present (user stories with P0/P1/P2 tags, affected surfaces, dependencies). The EA/GA tagging inside `tech-design.md` is a 15-minute annotation pass after the meeting — not a structural dependency. Running both speculatively means the joint meeting has concrete proposals to react to rather than starting from a blank slate.

---

#### Phase 3 — Functional Requirements (parallelized)

**Skills:** `/func-req --milestone ea` + `/func-req --milestone ga` — run simultaneously

1. Both milestones generated in parallel (they are already independent of each other).
2. AI runs a consistency check before presenting for review: verifies EA and GA stories don't overlap, GA doesn't repeat EA technical tasks, all EA blockers are flagged.
3. Eng Lead reviews both func-req files in a single combined pass.
4. Sets `status: approved` on both files.

**Human checkpoint:** Eng Lead sets `approved` on both func-req files.

---

#### Phase 4 — JIRA Ticket Creation (enhanced)

**Skills:** `/jira-tickets --milestone ea` + `/jira-tickets --milestone ga`

1. Creates Epic + Stories as before (one story per technical task).
2. If `retro.md` exists from a previous project run, AI applies calibration hints: flags stories that match historically oversized or under-specified patterns.
3. AI auto-sets `status: done` on func-req files after tickets are created.

**Human checkpoint:** Eng Lead opens board → starts coding.

---

#### Phase 5 — Feedback Loop (new)

**Skill:** `/retro --project <name> --jira <url-or-jql>`

1. Eng Lead provides a JIRA link or JQL filter for completed tickets (e.g., Epic link, sprint board URL, or `project = ORCA AND fixVersion = "EA"`)
2. AI pulls all tickets via JIRA MCP, reads `functional-requirements-ea.md` and `functional-requirements-ga.md`, and compares planned vs. actual:
   - Stories that were split or merged during implementation
   - Tickets flagged as too vague or too large during the sprint
   - Actual complexity vs. scope-propose estimate (P0 stories that ran long, P2 stories that were trivial)
   - Acceptance criteria that changed mid-sprint
3. Writes findings to `docs/projects/<name>/retro.md`.
4. Findings feed forward into future runs: scope-propose uses historical complexity signals; tech-design risk patterns updated; PRD template hints surfaced.
5. Eng Lead sets `status: approved` on `retro.md` to confirm findings are accurate before they influence future projects.

**Human checkpoint:** Eng Lead sets `approved` on `retro.md`.

---

### New Skills Required

| Skill | Command | Purpose |
|---|---|---|
| Challenge | `/challenge --project <name>` | AI devil's advocate on PRD before enrich — surfaces assumptions, missing personas, contradictions |
| Pre-Meeting Brief | `/pre-meeting-brief --project <name>` | 1-page decision summary: debatable scope items, EA blockers, open questions, risks. Auto-triggered after parallel AI analysis. |
| Retro | `/retro --project <name> --jira <url-or-jql>` | Post-delivery feedback loop — compares planned vs. actual JIRA outcomes, calibrates future runs |

### Modified Skills

| Skill | Change |
|---|---|
| `/prd --enrich` | After completion, auto-triggers `/scope-propose` and `/tech-design` in parallel |
| `/scope-propose` | Output appended to `prd.md` as a section instead of written to a separate file. Status vocabulary: `draft` / `approved` |
| `/tech-design` | Runs speculatively on full scope. EA/GA tags added post-meeting annotation pass. `repos-scanned` moved from frontmatter to an inline comment block. Status: `draft` / `approved` |
| `/func-req` | Both milestones run simultaneously. AI consistency check before presenting for review. |
| `/jira-tickets` | Reads `retro.md` for calibration hints if available. Auto-sets `done` on func-req files. |
| All skills | Status vocabulary unified to `draft` / `approved` / `done` |

---

### Role Boundaries

| Role | Owns | Never does |
|---|---|---|
| **AI** | Generates all artifacts · Challenges all decisions · Maintains cross-project memory via retro | Makes final decisions · Sets `approved` status |
| **PM** | Defines "what" · Responds to `/challenge` · Final say on scope decisions | Writes technical artifacts · Runs skills manually (except `/prd`) |
| **Eng Lead** | Validates technical accuracy · Approves artifacts · Confirms retro findings | Writes first drafts · Manually sequences pipeline steps |

---

## Data Flow — Status Progression

```
prd.md:                    draft → approved (after /challenge + PM review)
                           (scope proposal section appended by /scope-propose, same file)
tech-design.md:            draft → approved (after joint meeting + EA/GA tagging)
functional-requirements-*: draft → approved → done
retro.md:                  draft → approved
```

---

## Error Handling

- **Parallel drafts when enrich fails:** If `/prd --enrich` exits with an error, parallel tracks do not trigger. Error surfaces to eng lead.
- **Scope changes after tech design approval:** If scope decisions in the joint meeting require significant architectural changes (not just EA/GA tagging), tech design goes back to `draft`. Pre-meeting brief should flag high-risk scope items to reduce this.
- **JIRA MCP unavailable for retro:** `/retro` stops with a clear message. Does not fail silently or produce a partial retro.
- **Partial ticket creation:** Existing behavior preserved — stop and report partial state, don't retry automatically.

---

## Open Questions

- Should the pre-meeting brief be written to a file or just output to the terminal? Writing to a file allows async review before the meeting.
- How does cross-project retro memory work in practice — does it reference other `retro.md` files by path, or is there a summary index?

## Decisions

- `/challenge` runs automatically after `/prd` completes — lower friction, consistent with how `/prd --enrich` auto-triggers parallel drafts.

---

## Time Comparison

| | Current | Proposed |
|---|---|---|
| Human review cycles | 4 async + 2 meetings | 2 async + 1 joint meeting |
| Eng Lead PM time | ~1 hour | ~45 min (+ retro post-delivery) |
| Files to track | 5 per project | 4 per project (scope-proposal.md eliminated) |
| Status values to remember | 7 across 4 files | 3 across all files |
