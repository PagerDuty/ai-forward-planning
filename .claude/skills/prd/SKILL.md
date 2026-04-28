---
name: prd
description: "Use when a PM has a rough idea or a written PRD that needs to be structured into the standard format with P0/P1/P2-tagged user stories. Auto-discovers the source doc in the project folder. Use --approved to skip the challenge step."
---

# PRD Skill

Reads whatever the PM has — a full PRD, rough notes, or nothing — and produces a structured `prd.md`. Interviews only for sections that are missing or weak.

## Arguments

```
/prd --project <name>
/prd --project <name> --approved          # skip challenge step
/prd --project <name> --mocks <figma-url> # include Figma mocks
```

**Resolve project:** Use `--project <name>` if provided (writes to `.current-project`). Else read `.current-project`. If neither, stop: *"No project set. Run with `--project <name>` or set `.current-project`."*

---

## Steps

### Step 1: Find source doc

Scan `docs/projects/<name>/` for any `.md` file that is not a pipeline output (`prd.md`, `tech-design.md`, `functional-requirements-ea.md`, `functional-requirements-ga.md`).

- **One file found** → read it. This is the source doc.
- **Multiple files found** → ask: *"Found [file1, file2] — which one should I use?"*
- **No file found** → proceed with full interview (Step 2a).

**Check for design mocks:** If `--mocks <figma-url>` was provided, save it. Otherwise scan the source doc for any Figma URLs. If neither, ask once: *"Do you have Figma mocks? If yes, share the URL."* Optional — skip if none.

### Step 2: Assess and fill gaps

**If source doc found:** Read it and assess each required section:

| Section | Strong signal | Weak / missing |
|---------|--------------|----------------|
| Problem Statement | Clear problem, persona, why now | Vague, no persona, or missing |
| Goals | Measurable outcomes | Feature list, or missing |
| Non-Goals | Explicit exclusions | Missing |
| User Stories | Stories with priority signals | No stories, no priorities, or too vague |

For each **strong** section: extract and use as-is.
For each **weak or missing** section: ask a targeted question to fill it. Only ask what's needed — don't re-interview what's already clear.

**If no source doc (full interview):** Ask each question one at a time, waiting for the answer:

1. *"Who is the primary user? Describe their role and context."*
2. *"What does success look like 6 months after GA? Give 1–2 measurable outcomes."*
3. *"Walk me through the must-haves [P0], important-but-deferrable [P1], and nice-to-haves [P2]."*
4. *"What are the explicit non-goals — things NOT in scope?"*
5. *"Any known dependencies or open questions before development starts?"*
6. *"Any constraints — timeline, compliance, existing systems?"*
7. *"Does this involve AI/ML? If yes: model, data source, how quality is measured."*

Skip any question the PM's input already answers clearly.

### Step 3: Draft the PRD

Using the source doc + any interview answers, fill all sections of `template.md`:
- Apply P0/P1/P2 tags to every user story based on PM's language ("must/required/critical" → P0, "should/important" → P1, "nice to have/future" → P2). Hedged language ("maybe", "could", "might") → `[NEEDS TAG]`
- Assign each story a stable ID: `US-01 [P0] As a...`. IDs must be preserved in all downstream files.
- Leave sections 5-6 as: `<!-- Run /analyze --project <name> to populate this section -->`
- If AI/ML is involved, add placeholder under Open Questions: *"AI components involved — fill in model, data source, evaluation approach, and failure modes before tech design."*

### Step 4: Challenge the draft

**Skip this step if** `--approved` was passed, or the PM says anything like *"this is approved"*, *"skip challenge"*, *"looks good"*.

Otherwise, review the draft as a skeptical engineering lead. Check for:

1. **Unstated assumptions** — infra or behaviors taken for granted
2. **Missing personas** — user types touched by the feature with no story
3. **Story contradictions** — stories conflicting with each other or with non-goals
4. **Scope vagueness** — stories too broad to estimate
5. **Missing edge cases** — obvious failure modes not addressed

If issues found: present as a numbered list. Wait for PM response. Update the draft accordingly.
If no issues: note *"No challenges — PRD looks solid."* and proceed.

### Step 5: Write file

Write to `docs/projects/<name>/prd.md`:
```yaml
---
project: <name>
created: <today's date YYYY-MM-DD>
status: draft
design-mocks: <figma-url or blank>
---
```

### Step 6: Prompt PM

> "PRD saved to `docs/projects/<name>/prd.md`.
>
> Review the user stories and priority tags. When ready, set `status: approved` and run `/analyze --project <name>`."

---

## Notes

- Do not invent requirements — only use what came from the source doc or interview
- Do not pre-split stories into EA/GA — that is `/analyze`'s job
- Every user story must have a P0/P1/P2 tag and a stable US-XX ID before the file is written
