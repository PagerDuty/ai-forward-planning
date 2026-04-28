---
name: prd
description: Use when a PM has a rough idea or written PRD that needs to be structured into the standard format with P0/P1/P2-tagged user stories. Three modes: default (interview PM), --ingest (restructure existing PM doc), --enrich (codebase scan to fill engineering context).
---

# PRD Skill

Three modes depending on where the PM is in the process.

## Arguments

```
/prd --project <name>                                        # interview mode (PM needs help drafting)
/prd --ingest --project <name> --file <path>                 # restructure PM's existing doc
/prd --enrich --project <name>                               # codebase scan to fill sections 5-6
/prd --project <name> --mocks <figma-url>                    # interview mode with Figma mocks
/prd --ingest --project <name> --file <path> --mocks <url>   # ingest with Figma mocks
```

- `--project` — short slug for the product/feature (e.g., `audit-log`, `onboarding-flow`). Used as folder name under `docs/projects/`. **Optional if `.current-project` is set.**
- `--file` — path to PM's existing PRD document (required with `--ingest`)
- `--mocks` — Figma URL for design mocks (optional). Saved to `prd.md` frontmatter and used by all downstream skills. Can also be added later by editing `design-mocks:` in `prd.md` directly.

## Resolve project name

Run this before any mode:

1. If `--project <name>` is provided: use it. Write the value to `.current-project` at the repo root.
2. Else read `.current-project` from repo root — use the value if present.
3. Else run `git branch --show-current`, strip common prefixes (`feature/`, `fix/`, `feat/`, `bugfix/`) and leading ticket numbers (e.g., `ENG-123-`). If `docs/projects/<slug>/` exists, use it.
4. If none resolve: stop with — *"No project set. Provide `--project <name>` or run `echo '<name>' > .current-project` to set a default."*

When resolved from `.current-project` or git branch (not from `--project`), confirm before proceeding: *"Using project `<name>` — is that right?"*

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

**Check for design mocks:** If `--mocks <figma-url>` was provided, save it — it will be written to frontmatter in Step 4. If not provided, check if the PM's input contains any Figma URLs (figma.com/design/..., figma.com/file/...) and extract them. If neither, ask once: *"Do you have Figma mocks for this feature? If yes, share the URL and I'll use them throughout the pipeline."* This is optional — skip if the PM has no mocks yet.

### Step 2: Run the interview

Ask each question **one at a time**. Wait for the answer before asking the next.

**General questions (always ask all):**

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
- Assign each user story a stable ID in the format `US-01`, `US-02`, etc., prefixed before the P0/P1/P2 tag. Example: `US-01 [P0] As a compliance officer...`. This ID must be preserved unchanged in all downstream files.
- Do NOT pre-split into EA/GA — that is `/scope-propose`'s job
- Leave sections 5-6 as empty placeholders: write `<!-- Run /prd --enrich --project <name> to populate this section -->` as the only content in each
- If PM indicated AI/ML involvement, add an **AI Considerations** placeholder under Open Questions: "AI components involved — fill in model, data source, evaluation approach, and failure modes before tech design begins."

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

If a Figma URL was collected in Step 1, write it to `design-mocks`. Otherwise leave it blank.

### Step 5: Prompt PM

> "PRD drafted and saved to `docs/projects/<name>/prd.md`.
>
> Review the user stories and priority tags. When satisfied, set `status: approved` in `prd.md` (PM or Eng Lead can approve), then run `/prd --enrich --project <name>` for engineering context."
>
> Sections 5-6 are empty — run `/prd --enrich --project <name>` to fill them with codebase context before running `/scope-propose`. If you skip it, `/scope-propose` will offer to scan the codebase inline, but the highest-quality scope cut comes from a dedicated enrichment pass.

---

## Mode 2: Ingest (`/prd --ingest --project <name> --file <path>`)

Use when PM has already written a PRD. Restructures it — no interview.

### Steps

1. Read the file at `--file` path (any format — exported Google Doc, Notion, plain text, markdown). Also check for any Figma URLs (figma.com/design/..., figma.com/file/...) embedded in the source doc — if found, use them as `design-mocks` unless `--mocks` was explicitly provided. If `--mocks` was provided, that takes precedence.
2. Extract and map content to template sections 1-4: Problem Statement, Goals, Non-Goals, User Stories
   - Apply P0/P1/P2 tags to every user story based on PM's language:
     - "must", "required", "critical", "blocker" → `[P0]`
     - "should", "important", "want" → `[P1]`
     - "nice to have", "ideally", "future", "consider" → `[P2]`
     - **Hedged language → flag for PM review, do not assign a tag.** If the PM used tentative framing ("maybe", "rough idea", "could", "might", "I think", "perhaps", "not sure yet", "we could consider"), the story's priority is genuinely unclear — append `[NEEDS TAG - PM to assign P0/P1/P2]` rather than guessing. The cost of a wrong tag propagates through scope, tech design, and JIRA; it is better to surface the ambiguity than to resolve it silently.
     - Stories with no priority signal at all → also flag with `[NEEDS TAG - PM to assign P0/P1/P2]`
   - Assign each user story a stable ID in the format `US-01`, `US-02`, etc., prefixed before the priority tag. Example: `US-01 [P0] As a compliance officer...`. This ID must be preserved unchanged in all downstream files.
   - If the source doc mentions AI/ML components, add an **AI Considerations** placeholder under Open Questions: "AI components detected — fill in model, data source, evaluation approach, and failure modes before tech design begins."
3. Extract Success Metrics and Open Questions verbatim from the source doc. Do not synthesize, infer, or invent entries for these sections:
   - If Success Metrics are absent from the source doc, write: `<!-- Success Metrics not found in source doc — PM to add before approving -->`
   - If Open Questions are absent, write: `<!-- Open Questions not found in source doc — PM to add before approving -->`
   - The only permitted addition is the AI Considerations placeholder (step 2 above)
4. Flag any missing required section — do not invent content:
   - *"Non-goals not found in PM's doc — please add before proceeding"*
   - **If User Stories are missing or cannot be extracted from the source doc, stop and do not write the file.** Output: *"User Stories section not found in the provided document. This section is required — ask the PM to add it before re-running ingest."*
5. Do NOT pre-split stories into EA/GA
6. Leave the Affected Surfaces and Dependencies & Constraints sections as empty placeholders: write `<!-- Run /prd --enrich --project <name> to populate this section -->` as the only content in each
7. Write `docs/projects/<name>/prd.md` with `status: draft` and `design-mocks: <url or blank>`

### What it does NOT do
- Interview the PM
- Invent requirements, metrics, or questions not in the source doc
- Populate Affected Surfaces or Dependencies & Constraints

### Prompt after completion

> "PRD ingested and saved to `docs/projects/<name>/prd.md`.
>
> Review the user stories and priority tags — flag any that were tagged automatically and need sign-off. When satisfied, set `status: approved` in `prd.md` (PM or Eng Lead can approve), then run `/prd --enrich --project <name>` for engineering context."
>
> Sections 5-6 are empty — run `/prd --enrich --project <name>` to fill them with codebase context before running `/scope-propose`. If you skip it, `/scope-propose` will offer to scan the codebase inline, but the highest-quality scope cut comes from a dedicated enrichment pass.

---

## Mode 3: Enrich (`/prd --enrich --project <name>`)

Engineering lead runs this after PM's PRD is confirmed. Fills technical context via codebase scan.

**Precondition:** `status: draft`, `approved`, or `scope-finalized` — warn and stop if none match. Re-running `--enrich` is safe at any status — it only updates sections 5-6 and does not change the status.

### Steps

1. Read `docs/projects/<name>/prd.md` — focus on user stories (section 4) and goals (section 2). Check `design-mocks` in frontmatter.

1a. **Fetch design mocks (if present):** If `design-mocks` is set and non-empty, use `mcp__claude_ai_Figma__get_design_context` to fetch the Figma file. Extract: screens, states, UI components, flows, and interaction notes. Hold this context for step 5 — it will inform which surfaces are affected and surface constraints the codebase scan should verify.

2. Verify GitHub MCP is configured and accessible. If not, switch to **manual fallback mode** — do not stop:
   > "GitHub MCP is not available, so I can't scan the codebase automatically. Two options:
   > - **Configure GitHub MCP** in your MCP settings and re-run `/prd --enrich` for codebase-grounded output.
   > - **Proceed manually**: describe the affected surfaces and constraints for each story and I'll fill sections 5-6 from your input. All entries will be tagged `[MANUAL - unverified]` so reviewers know they weren't codebase-derived.
   >
   > Proceed with manual input? (yes/no)"

   If **yes**: Ask the eng lead: *"List all repos and code areas involved — include every repo touched by any story, especially cross-team services (e.g. AI backends, shared libraries, separate frontend repos). For each user story, which areas does it touch and are there any known dependencies or constraints?"* Use their answers to fill sections 5-6, tagging every entry `[MANUAL - unverified]`. Skip steps 3-4 (no MCP scan). Continue to step 5.
   If **no**: stop.

3. **Ask the eng lead which repos to scan before searching:**
   > "Which GitHub repos contain the code for this feature? Provide as `org/repo`. Include every repo touched by any user story — cross-repo dependencies (separate services, shared libraries, AI/ML backends) are easy to miss and won't be found automatically.
   >
   > Example: `acme/workflow-engine, acme/ai-workflows, acme/pd-frontend`"

   Wait for the response. Use the provided list as the scan scope. If the eng lead is unsure of a repo for a specific story (e.g. "AI recommendations live somewhere but I don't know where"), note it as an unresolved cross-team dependency in Section 6 — do not guess or skip it.

4. Scan each provided repo via GitHub MCP, guided by the story descriptions
5. Fill **Section 5 — Affected Surfaces**: for each surface, identify which stories touch it and whether the pattern already exists in the codebase
6. Fill **Section 6 — Dependencies & Constraints**: flag missing indexes, services that need changes, patterns that do/don't exist, cross-team dependencies, open questions from the scan
7. For each finding include a one-line reason and source file location:
   ```
   - `events` table has no index on `user_id` — queries will be slow at scale
     (found: schema/events.sql, no index defined)
   - Export pattern already exists — reuse recommended
     (found: app/services/exports/csv_export.rb)
   - Auth team dependency: token → user resolution not implemented
     (flagged: stories mention "who accessed" but no resolution logic found)
   ```
8. Update `prd.md` in place — do not modify sections 1-4. Add `<!-- repos-scanned: [org/repo, ...] -->` as the first line of Section 5, listing every repo that was searched.
9. **Technical feasibility check:** After filling sections 5-6, scan for any findings that may block or fundamentally reshape the feature. If any of the following are present, surface a summary before proceeding:
   - A P0 story depends on a service or API that doesn't exist and requires a separate team to build
   - A pattern assumed in the PRD (e.g., "extend the existing X system") does not exist in the codebase
   - A dependency (DB schema, third-party service) would require more than trivial changes to accommodate the stories
   Output as: `⚠️ Feasibility flag: <finding> — recommend eng lead reviews before meeting.` These are informational — they do not stop the auto-trigger.

### After completion: next steps

After sections 5-6 are written, check `status` in `prd.md` and prompt accordingly:

**If `status: draft`:**
> "Enrichment complete — sections 5-6 filled. Review any feasibility flags above.
>
> Approval is required before scope analysis can run. PM or Eng Lead can approve — review the PRD and set `status: approved` in `prd.md`.
>
> Once approved, run: `/scope-propose --project <name>` — scores each story and appends a Scope Proposal and Meeting Brief to `prd.md`."

**If `status: approved`:**
> "Enrichment complete — sections 5-6 filled. Review any feasibility flags above.
>
> Run: `/scope-propose --project <name>` — scores each story, appends a Scope Proposal and Meeting Brief to `prd.md`. Share `prd.md` with the PM before the joint meeting.
>
> After the joint meeting:
> - Record scope decisions in the Scope Proposal section of `prd.md`
> - Set `status: scope-finalized` in `prd.md` (Eng Lead, after PM sign-off)
> - Run `/tech-design --project <name>`
> - Set `status: approved` on `tech-design.md`, then run `/func-req`"

---

## Status Gates

`prd.md` uses a single `status` field with three values:

| Value | Set by | Means |
|---|---|---|
| `draft` | AI (automatic) | PRD written, not yet PM-approved |
| `approved` | PM or Eng Lead (manual) | Signed off — enrichment and scope can run |
| `scope-finalized` | Eng Lead (manual, post-meeting) | Scope locked — unlocks `/tech-design` and `/func-req` |

Downstream skills check `status`:
- `/prd --enrich` runs at any status (re-run safe); auto-trigger only fires when `status: approved`
- `/scope-propose` requires `status: approved`
- `/tech-design` requires `status: scope-finalized`
- `/func-req` requires `status: scope-finalized`

## Notes

- Do not pre-split stories into EA/GA — that is `/scope-propose`'s job
- Do not invent requirements — only use what came from PM's input or interview
- Every user story must have a P0/P1/P2 tag before the file is considered complete
- Every user story must have a stable US-XX ID that is preserved across all downstream files
- If the PM's draft already answers an interview question clearly, skip that question
