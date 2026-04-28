---
name: scope-propose
description: "Use when a PRD is approved and you need to propose an EA/GA scope cut for PM and engineering lead to align on before tech design begins. Uses judgment to assign stories to EA/GA and appends a Scope Proposal + Meeting Brief to prd.md with debatable items flagged."
---

# Scope Propose Skill

Reads the enriched PRD and proposes an EA/GA scope cut — so the alignment meeting reacts to a concrete proposal rather than starting from a blank slate.

## Arguments

```
/scope-propose --project <name>
```

- `--project` — slug matching the folder under `docs/projects/`. **Optional if `.current-project` is set.**

## Resolve project name

1. If `--project <name>` is provided: use it. Write the value to `.current-project` at the repo root.
2. Else read `.current-project` from repo root — use the value if present.
3. Else run `git branch --show-current`, strip common prefixes (`feature/`, `fix/`, `feat/`, `bugfix/`) and leading ticket numbers (e.g., `ENG-123-`). If `docs/projects/<slug>/` exists, use it.
4. If none resolve: stop with — *"No project set. Provide `--project <name>` or run `echo '<name>' > .current-project` to set a default."*

When resolved from `.current-project` or git branch (not from `--project`), confirm before proceeding: *"Using project `<name>` — is that right?"*

**Precondition:** `prd.md` must have `status: approved` — warn and stop if not met.

---

## Steps

### Step 0: Prepare context

Read `docs/projects/<name>/prd.md` and verify `status: approved` — warn and stop if not set.

**Fetch design mocks (if present):** Check `design-mocks` in frontmatter. If set and non-empty, use `mcp__claude_ai_Figma__get_design_context` to fetch the Figma file before doing anything else. Extract: number of screens per user story, interaction complexity (simple toggle vs. multi-state flows), edge-case states (empty states, error states, loading states), and any screens that look clearly MVP vs. polished. Hold this context — it is a direct complexity signal for Step 1.

Check whether sections 5 (Affected Surfaces) and 6 (Dependencies) are populated. If either section contains only empty dashes, blank lines, HTML comments, or the placeholder text "Run `/prd --enrich` to populate", treat it as empty and offer to scan inline:

> "Sections 5-6 are empty — engineering context is missing. I can scan the codebase via GitHub MCP now (same output as `/prd --enrich`) or proceed without it.
>
> Scan now? (yes/no — 'no' proceeds without codebase context; more stories will fall into Needs Discussion)"

**If yes and GitHub MCP is available:**
1. Ask: *"Which repos contain the code for this feature? Provide as `org/repo`."*
2. Scan those repos — focus on areas relevant to the user stories in section 4. Extract affected files, existing patterns, schema, and cross-team dependencies.
3. Write sections 5-6 in `prd.md` using the same format as `/prd --enrich` (file locations, one-line findings). Tag everything `[SCAN - scope-propose]` so reviewers know it was done inline.
4. Surface any `⚠️ Feasibility flag` findings before continuing.
5. Continue to Step 1 with full engineering context.

**If yes and GitHub MCP is NOT available:**
> "GitHub MCP is not available. Proceed with manual input? (yes/no)
> If yes: describe the affected surfaces and constraints per story — I'll fill sections 5-6, tagged `[MANUAL - unverified]`.
> If no: proceed without codebase context."

**If no:** continue with complexity and risk signals defaulting to "unknown".

### Step 1: Read and assess all user stories

Read `docs/projects/<name>/prd.md` in full — user stories (section 4), affected surfaces (section 5), and dependencies (section 6).

For each story, use judgment to assess its fit for EA vs GA based on four inputs:
- **P-level** (from PM) — the primary signal. P0 strongly favors EA, P2 defaults to GA, P1 requires weighing the other signals.
- **Complexity** (from section 5) — how many surfaces does it touch? Does the pattern already exist in the codebase, or is this net-new?
- **Risk** (from section 6) — does this story have an unresolved external dependency or constraint that could block EA?
- **Design complexity** (from Figma mocks, if fetched in Step 0) — how many screens and states does this story require? A story with 1-2 simple screens favors EA; a story with many screens, complex interaction flows, or unfinished/placeholder screens in the mocks favors GA or Needs Discussion.

P1 stories where these signals conflict are candidates for **Needs Discussion**. Flag them with your reasoning — EA case, GA case, and a recommendation. When mocks were used, include a brief note in the reasoning (e.g., "Mocks show 4 edge-case states — higher design complexity than P1 signal suggests").

If sections 5-6 are empty, note it in the reasoning for each story and flag more items as Needs Discussion rather than guessing.

### Step 2: Append Scope Proposal section to `prd.md`

Append the following to the bottom of `docs/projects/<name>/prd.md`. Do not create a separate file.

```markdown
---

## Scope Proposal

Generated: <date>

### EA Scope

✅ US-01 [P0] As a <persona>, I want <capability>...
   Reason: <one line — priority signal + complexity signal>

✅ US-04 [P1] As a <persona>, I want <capability>...
   Reason: <one line>

---

### GA Scope

⏩ US-03 [P2] As a <persona>, I want <capability>...
   Reason: <one line — why deferred>

⏩ US-05 [P2] As a <persona>, I want <capability>...
   Reason: P2 priority — deferred by default

---

### Needs Discussion (<N> items)

<!-- If N=0, write: "All stories have clear signals — no alignment discussion needed." -->

These stories have conflicting signals. PM and engineering lead should decide.

⚡ US-02 [P0] As a <persona>, I want <capability>...
   EA case: <reason it could go to EA>
   GA case: <reason it should be deferred>
   Recommendation: <EA or GA> — <one sentence rationale>

---

### Summary

EA: <N> stories
GA: <N> stories
Needs discussion: <N> items
```

### Step 3: Append Meeting Brief section to `prd.md`

Immediately after the Scope Proposal section, append a Meeting Brief. This is a 1-page summary of the items requiring a human decision — everything else is pre-proposed. Keep it to the decision points only; do not repeat content from the Scope Proposal.

```markdown
---

## Meeting Brief

**Purpose:** 20-minute joint meeting agenda. React to concrete proposals — don't start from scratch.

### Agenda
1. Scope decisions — resolve Needs Discussion items (~15 min)
2. Risk items — decide whether to buffer or descope (~5 min)

### Scope Decisions Required (<N> items)

<!-- One block per Needs Discussion story. If N=0, write: "No decisions required — all stories have clear signals." -->

**US-XX [P0] <story summary>**
- EA case: <reason>
- GA case: <reason>
- Recommendation: <EA or GA>
- Decision: ___

### EA Blockers (<N> items)

<!-- Stories that cannot ship EA until an external dependency is resolved. If none, omit section. -->

- **<Blocker>** — <what is blocked> — Owner: ___

### Risk Flags (<N> items)

<!-- Elevated risk patterns: high-complexity story in EA, unresolved dep assigned EA, strong conflict. If none, omit. -->

- **<Risk>** — <why> — Mitigation: ___

### After This Meeting

- [ ] Eng Lead: record scope decisions in the Scope Proposal section of `prd.md` (move ⚡ items to ✅ EA or ⏩ GA)
- [ ] Eng Lead: set `status: scope-finalized` in `prd.md` after PM sign-off
- [ ] Eng Lead: run `/tech-design --project <name>`
```

### Step 4: Prompt PM + engineering lead

**Check affected surfaces** (from sections 5-6): if all surfaces are UI/frontend-only — no API, DB, backend workers, or external service changes — add a note to the prompt below:
> "**Note:** Affected surfaces appear UI-only. You may be able to skip `/tech-design` — `/func-req` will confirm and offer a UX-only path if `tech-design.md` is absent."

**Branch on Needs Discussion count:**

**If N=0 (all stories have clear EA/GA assignments):**
> "Scope proposal appended to `docs/projects/<name>/prd.md`.
>
> All stories have clear signals — no alignment meeting needed.
>
> Review the Scope Proposal in `prd.md`. If the assignments look right:
> 1. Get PM sign-off on the scope
> 2. Set `status: scope-finalized` in `prd.md` (Eng Lead, after PM sign-off)
> 3. Run `/tech-design --project <name>`"
>
> [Add UI-only note here if applicable]

**If N>0 (items require a decision):**
> "Scope proposal and meeting brief appended to `docs/projects/<name>/prd.md`.
>
> Share `prd.md` with the PM — the **Meeting Brief** section at the bottom is the agenda. The **Needs Discussion** items are the only things requiring a decision; everything else is pre-proposed.
>
> After the joint meeting: move resolved ⚡ items to EA or GA scope in `prd.md`, then set `status: scope-finalized` (Eng Lead, after explicit PM sign-off). That unlocks `/tech-design`."
>
> [Add UI-only note here if applicable]

---

## After the Meeting (applies when N>0)

Engineering lead:
1. Opens `prd.md` and records final decisions on debatable items in the Scope Proposal section — move resolved ⚡ items to EA or GA scope, changing emoji to ✅ or ⏩
2. Sets `status: scope-finalized` in `prd.md` frontmatter (after explicit PM sign-off)

`/func-req` checks for `status: scope-finalized` on `prd.md` before running.

---

## Notes

- Do not generate tech tasks or implementation details — this is scope only
- The proposal is a starting point, not a final decision — PM and engineering lead override freely
- If sections 5-6 are empty, scope-propose offers to run the codebase scan inline (same as `/prd --enrich`). Running `/prd --enrich` separately first is still valid — it just means the scan is already done when scope-propose runs.
- When N=0 (Needs Discussion is empty), no meeting is needed — eng lead sets `status: scope-finalized` directly after PM sign-off
- Stories that depend on each other should be grouped and called out explicitly in the proposal
- When the eng lead records decisions on Needs Discussion items after the meeting, they should move the story entry to the EA Scope or GA Scope section and change its emoji from `⚡` to `✅` or `⏩` accordingly — this is what `/func-req` uses to determine milestone assignment
