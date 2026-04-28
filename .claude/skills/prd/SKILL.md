---
name: prd
description: "Use when a PM has a rough idea or written PRD that needs to be structured into the standard format with P0/P1/P2-tagged user stories. Two modes: default (interview PM), --ingest (restructure existing PM doc)."
---

# PRD Skill

Two modes depending on where the PM is in the process.

## Arguments

```
/prd --project <name>                                        # interview mode (PM needs help drafting)
/prd --ingest --project <name> --file <path>                 # restructure PM's existing doc
/prd --project <name> --mocks <figma-url>                    # interview mode with Figma mocks
/prd --ingest --project <name> --file <path> --mocks <url>   # ingest with Figma mocks
```

- `--project` — short slug for the product/feature (e.g., `audit-log`, `onboarding-flow`). Used as folder name under `docs/projects/`. **Optional if `.current-project` is set.**
- `--file` — path to PM's existing PRD document (required with `--ingest`)
- `--mocks` — Figma URL for design mocks (optional). Saved to `prd.md` frontmatter and used by all downstream skills.

**Resolve project:** `--project <name>` → write to `.current-project`. Else read `.current-project`. Else derive from git branch (strip `feature/`, `fix/`, `feat/`, `bugfix/`, leading ticket numbers like `ENG-123-`). If resolved from context, confirm first. If unresolvable, stop.

---

## Mode 1: Interview (`/prd --project <name>`)

Use when PM has a rough idea but hasn't written a PRD yet.

### Step 1: Collect input

Accept input in any form:
- **Pasted draft** — PM pastes text from a doc, email, or notes
- **File reference** — PM provides a file path; read the file
- **Inline description** — PM describes the idea directly

If no input is provided, ask: *"Share your rough draft or describe the feature — paste a doc, give a file path, or just describe it."*

Summarize what you understood in 2–3 sentences and confirm with the PM before proceeding.

**Check for design mocks:** If `--mocks <figma-url>` was provided, save it. If not, scan the PM's input for Figma URLs (figma.com/design/..., figma.com/file/...). If neither, ask once: *"Do you have Figma mocks? If yes, share the URL and I'll use them throughout the pipeline."* Optional — skip if none.

### Step 2: Run the interview

Ask each question **one at a time**. Wait for the answer before asking the next. Skip any question the PM's input already answers clearly.

1. *"Who is the primary user for this feature? Describe their role and the context in which they'd use it."*
2. *"What does success look like 6 months after GA? Give 1–2 measurable outcomes."*
3. *"Walk me through the must-haves [P0], what's important but could slip [P1], and what's nice to have [P2]."*
4. *"What are the explicit non-goals — things we are NOT building in this scope?"*
5. *"Any known dependencies (other teams, systems, APIs) or open questions before development starts?"*
6. *"Any constraints — timeline, compliance, existing systems that must be preserved?"*
7. *"Does this feature involve AI or ML components? If yes, describe briefly — model, data source, how quality will be measured."*

### Step 3: Draft the PRD

Using input + interview answers, fill all sections of `template.md`:
- Apply P0/P1/P2 tags to every user story
- Assign each user story a stable ID: `US-01`, `US-02`, etc., prefixed before the priority tag. Example: `US-01 [P0] As a compliance officer...`. This ID must be preserved unchanged in all downstream files.
- Do NOT pre-split into EA/GA — that is `/analyze`'s job
- Leave sections 5-6 as empty placeholders: `<!-- Run /analyze --project <name> to populate this section -->`
- If PM indicated AI/ML involvement, add an **AI Considerations** placeholder under Open Questions: "AI components involved — fill in model, data source, evaluation approach, and failure modes before tech design begins."

### Step 3.5: Challenge the draft

Before writing the file, review the draft as a skeptical engineering lead. Check for:

1. **Unstated assumptions** — infra or behaviors taken for granted (existing systems that may not exist, user habits that may not hold, team capacity)
2. **Missing personas** — user types who interact with the feature but have no user story
3. **Story contradictions** — stories that conflict with each other or with the non-goals
4. **Scope vagueness** — stories where "I want X" is too broad to estimate (could be 2 days or 2 months)
5. **Missing edge cases** — obvious failure modes (empty states, errors, scale) not addressed in any story

If issues found: present as a numbered list. For each item, state the issue and why it matters. Wait for PM to respond. Update the draft accordingly; confirm intentional deferrals in Open Questions.

If no issues: note *"No challenges — PRD looks solid."* and proceed.

### Step 4: Write file

Write to `docs/projects/<name>/prd.md`:
```yaml
---
project: <name>
created: <today's date YYYY-MM-DD>
status: draft
design-mocks: <figma-url or blank>
---
```

### Step 5: Prompt PM

> "PRD drafted and saved to `docs/projects/<name>/prd.md`.
>
> Review the user stories and priority tags. When satisfied, set `status: approved` in `prd.md` (PM or Eng Lead can approve), then run `/analyze --project <name>` for codebase analysis and scope proposal."

---

## Mode 2: Ingest (`/prd --ingest --project <name> --file <path>`)

Use when PM has already written a PRD. Restructures it — no interview.

### Steps

1. Read the file at `--file` path (any format — exported Google Doc, Notion, plain text, markdown). Check for any Figma URLs embedded in the source doc. If `--mocks` was explicitly provided, that takes precedence.
2. Extract and map content to template sections 1-4: Problem Statement, Goals, Non-Goals, User Stories
   - Apply P0/P1/P2 tags to every user story based on PM's language:
     - "must", "required", "critical", "blocker" → `[P0]`
     - "should", "important", "want" → `[P1]`
     - "nice to have", "ideally", "future", "consider" → `[P2]`
     - **Hedged language → flag for PM review.** If the PM used tentative framing ("maybe", "could", "might", "I think", "perhaps", "not sure yet"), append `[NEEDS TAG - PM to assign P0/P1/P2]` rather than guessing. Wrong tags propagate through scope, tech design, and JIRA.
     - Stories with no priority signal at all → also flag with `[NEEDS TAG]`
   - Assign each user story a stable ID: `US-01`, `US-02`, etc. This ID must be preserved unchanged in all downstream files.
   - If the source doc mentions AI/ML components, add an **AI Considerations** placeholder under Open Questions.
3. Extract Success Metrics and Open Questions verbatim from the source doc. Do not synthesize or invent entries:
   - If Success Metrics are absent: `<!-- Success Metrics not found in source doc — PM to add before approving -->`
   - If Open Questions are absent: `<!-- Open Questions not found in source doc — PM to add before approving -->`
4. Flag any missing required section — do not invent content:
   - *"Non-goals not found in PM's doc — please add before proceeding"*
   - **If User Stories are missing or cannot be extracted, stop and do not write the file.** Output: *"User Stories section not found. This section is required — ask the PM to add it before re-running ingest."*
5. Leave sections 5-6 as empty placeholders: `<!-- Run /analyze --project <name> to populate this section -->`
6. Write `docs/projects/<name>/prd.md` with `status: draft` and `design-mocks: <url or blank>`

### Step 2.5: Challenge the ingested PRD

After extracting and mapping content, challenge the draft using the same criteria as Mode 1 Step 3.5. Present issues, wait for PM response, update the draft accordingly before writing the file.

### After completion

> "PRD ingested and saved to `docs/projects/<name>/prd.md`.
>
> Review the user stories and priority tags — flag any that were tagged automatically and need sign-off. When satisfied, set `status: approved` in `prd.md`, then run `/analyze --project <name>` for codebase analysis and scope proposal."

---

## Status Gates

`prd.md` uses a `status` field with two values:

| Value | Set by | Means |
|---|---|---|
| `draft` | AI (automatic) | PRD written, not yet approved |
| `approved` | PM or Eng Lead (manual) | Signed off — `/analyze` and downstream skills can run |

Downstream skills check:
- `/analyze` requires `status: approved`
- `/tech-design` requires `status: approved` + Scope Proposal section with no ⚡ items
- `/func-req` requires `status: approved` + Scope Proposal section with no ⚡ items

## Notes

- Do not pre-split stories into EA/GA — that is `/analyze`'s job
- Do not invent requirements — only use what came from PM's input or interview
- Every user story must have a P0/P1/P2 tag before the file is considered complete
- Every user story must have a stable US-XX ID that is preserved across all downstream files
