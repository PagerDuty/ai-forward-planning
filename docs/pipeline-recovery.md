# Pipeline Recovery Runbook

Use this when a skill fails mid-run, files are in an inconsistent state, or the pipeline needs to be restarted from a checkpoint.

---

## How to Diagnose the Current State

Check `status` in `docs/projects/<name>/prd.md` frontmatter:

| Value | Means |
|---|---|
| `draft` | PRD written, not PM-approved; enrich can run but scope-propose is blocked |
| `approved` | PM approved; ready for enrich + scope-propose |
| `scope-finalized` | Scope locked post-meeting; unlocks `/tech-design` and `/func-req` |

Check `docs/projects/<name>/tech-design.md` frontmatter: `status: draft` or `status: approved`.

Check `docs/projects/<name>/functional-requirements-<ea|ga>.md` frontmatter: `status: draft`, `approved`, or `done`.

---

## Failure Scenarios

### Scenario 1: `/prd --enrich` ran but scope-propose did not append

**Symptom:** `prd.md` sections 5-6 are filled, but no Scope Proposal section at bottom.

**Recovery:**
```
/scope-propose --project <name>
```
Re-running scope-propose is safe — it appends only if the section doesn't exist.

---

### Scenario 2: scope-propose completed but tech-design did not run

**Symptom:** `prd.md` has a Scope Proposal section and `status: scope-finalized`, but `tech-design.md` is missing.

**Recovery:**
```
/tech-design --project <name>
```
Tech-design reads `prd.md` including the Scope Proposal. Running it standalone is safe.

---

### Scenario 3: `status: scope-finalized` set but ⚡ items still unresolved in Scope Proposal

**Symptom:** `/func-req` stops with "Story US-XX is still listed under Needs Discussion."

**Recovery:**
1. Eng Lead opens `prd.md` Scope Proposal section.
2. For each ⚡ story: move it to EA Scope (✅) or GA Scope (⏩) based on post-meeting decision.
3. Re-run `/func-req --project <name> --milestone ea` (or `ga`).

---

### Scenario 4: tech-design.md approved but has empty sections

**Symptom:** `/func-req` generates stories with `[UNVERIFIED]` tasks or thin acceptance criteria.

**Recovery:**
1. Eng Lead opens `tech-design.md`.
2. Fills in the empty sections (API Contracts, DB Schema, Component Changes) based on codebase scan or design discussion.
3. Re-run `/func-req --project <name> --milestone <ea|ga>` — it will overwrite the previous draft.

---

### Scenario 5: GitHub MCP unavailable during `/prd --enrich`

**Symptom:** `/prd --enrich` offers a choice between manual fallback and stopping.

**Recovery — Option A (recommended):** Configure GitHub MCP in your Claude Code MCP settings, then re-run `/prd --enrich`. Re-running is safe — it only updates sections 5-6 and does not modify sections 1-4.

**Recovery — Option B:** Choose the manual fallback when prompted. Describe the affected surfaces and constraints for each story. Entries will be tagged `[MANUAL - unverified]`. Run `/scope-propose` after.

---

### Scenario 6: GitHub MCP unavailable during `/tech-design`

**Symptom:** `/tech-design` cannot scan repos — GitHub MCP is required and has no fallback.

**Recovery:**
1. Configure GitHub MCP in your Claude Code MCP settings.
2. Re-run `/tech-design --project <name>`.

Alternatively: manually write `tech-design.md` using the template structure and set `status: approved` directly. `/func-req` will use whatever is in the file.

---

### Scenario 7: `/func-req` generated but EA and GA files have overlapping stories

**Symptom:** The same US-XX appears in both `functional-requirements-ea.md` and `functional-requirements-ga.md`.

**Recovery:**
1. Identify which milestone the story belongs to (check the finalized Scope Proposal in `prd.md`).
2. Remove the story from the incorrect milestone file manually.
3. Re-run the affected milestone: `/func-req --project <name> --milestone <ea|ga>` — it will overwrite the file and re-run the consistency check.

Note: if re-running EA after GA already exists, the consistency check will be skipped for EA (it only runs for GA). Manually verify no overlap after re-running EA.

---

### Scenario 8: EA blocker flag is stale (dependency resolved but flag persists in JIRA)

**Symptom:** A JIRA story is marked `⚠️ EA blocker` but the blocking dependency has been delivered.

**Recovery:**
In `functional-requirements-ea.md`, find the task and update the flag:
```
~~⚠️ EA blocker~~ Resolved: <YYYY-MM-DD> — <one line: what was delivered / who delivered it>
```
Update the corresponding JIRA story description or comment to note the resolution.

---

## Pipeline State Checklist (quick audit)

Run this when you need to verify a project's pipeline state before a meeting or handoff:

- [ ] `prd.md` — `status: approved`?
- [ ] `prd.md` — Scope Proposal section present with no remaining ⚡ items?
- [ ] `prd.md` — `status: scope-finalized`?
- [ ] `tech-design.md` — `status: approved`?
- [ ] `tech-design.md` — All sections populated (API Contracts, DB Schema, Components)?
- [ ] `tech-design.md` — EA/GA tags filled in on all sections?
- [ ] `functional-requirements-ea.md` — `status: approved`?
- [ ] `functional-requirements-ga.md` — `status: approved` (if GA scope exists)?
- [ ] No `⚠️ EA blocker` flags on resolved dependencies?
