---
name: analyze
description: "Run after PM approves prd.md. Scans the codebase to fill sections 5-6, proposes EA/GA scope cut, and appends Scope Proposal + Meeting Brief to prd.md in one pass. Replaces /prd --enrich and /scope-propose."
---

# Analyze Skill

Fills engineering context and proposes EA/GA scope in one pass. Run after PM sets `status: approved` on `prd.md`, before tech design.

## Arguments

```
/analyze --project <name>
```

**Resolve project:** Use `--project <name>` if provided (writes to `.current-project`). Else read `.current-project`. If neither, stop: *"No project set. Run with `--project <name>` or set `.current-project`."*

**Precondition:** `prd.md` must have `status: approved`.

---

## Steps

### Step 1: Read PRD

Read `docs/projects/<name>/prd.md` in full. Verify `status: approved` — stop if not. Check `design-mocks` in frontmatter.

**Fetch design mocks (if present):** If `design-mocks` is set and non-empty, use `mcp__claude_ai_Figma__get_design_context`. Extract: screens per user story, interaction complexity (simple toggle vs. multi-state flows), edge-case states (empty, error, loading), MVP vs. polished signals. Write a `### Design Mocks Summary` subsection into section 5 when filling it in Step 3 — downstream skills (`/tech-design`, `/func-req`) read from there instead of re-fetching.

### Step 2: Scan codebase

Ask: *"Which GitHub repos contain the code for this feature? Provide as `org/repo`. Include every repo touched by any user story — cross-repo dependencies (separate services, shared libraries, AI/ML backends) are easy to miss."*

Wait for the response. Scan each repo via GitHub MCP, guided by the user story descriptions. Focus on:
- Route/controller files for API surfaces
- Schema/migration files for DB surfaces
- Component files for UI surfaces
- Worker/job files for background processing
- Service boundary definitions and integration points

For each finding, record a one-line reason and source file location:
```
- `events` table has no index on `user_id` — queries will be slow at scale
  (found: schema/events.sql, no index defined)
- Export pattern already exists — reuse recommended
  (found: app/services/exports/csv_export.rb)
- Auth team dependency: token → user resolution not implemented
  (flagged: stories mention "who accessed" but no resolution logic found)
```

If the engineering lead cannot identify a repo for a surface, note it as "Repo unknown — manual identification required" in Section 6.

### Step 3: Write sections 5-6 to `prd.md`

Update `docs/projects/<name>/prd.md` sections 5-6 in place. Do not modify sections 1-4.

Add `<!-- repos-scanned: [org/repo, ...] -->` as the first line of Section 5.

**Technical feasibility check:** After filling sections 5-6, scan for findings that may block or fundamentally reshape the feature:
- A P0 story depends on a service or API that doesn't exist and requires a separate team to build
- A pattern assumed in the PRD does not exist in the codebase
- A dependency would require more than trivial changes

Output as: `⚠️ Feasibility flag: <finding> — recommend eng lead reviews before proceeding.` These are informational — they do not stop execution.

### Step 4: Score and propose scope

For each user story, assess EA vs GA based on four signals:

- **P-level** — primary signal. P0 strongly favors EA. P2 defaults to GA. P1 requires weighing the other signals.
- **Complexity** — how many surfaces does it touch? Does the pattern already exist, or is it net-new?
- **Risk** — unresolved external dependencies that could block EA.
- **Design complexity** (from Figma, if fetched) — screens and states per story. Simple = favors EA. Many states or complex flows = favors GA or Needs Discussion.

P1 stories where signals conflict → **Needs Discussion**. Flag with EA case, GA case, and a recommendation. If Figma was used, include a brief note (e.g., "Mocks show 4 edge-case states — higher design complexity than P1 signal suggests").

### Step 5: Append Scope Proposal to `prd.md`

Append to the bottom of `docs/projects/<name>/prd.md`:

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

---

### Needs Discussion (<N> items)

<!-- If N=0, write: "All stories have clear signals — no discussion needed." -->

⚡ US-02 [P1] As a <persona>, I want <capability>...
   EA case: <reason>
   GA case: <reason>
   Recommendation: <EA or GA> — <one sentence rationale>

---

### Summary

EA: <N> stories
GA: <N> stories
Needs discussion: <N> items
```

### Step 6: Prompt

**UI-only check:** If all affected surfaces are UI/frontend-only (no API, DB, backend workers, or external service changes), add: *"Affected surfaces appear UI-only. You may be able to skip `/tech-design` — `/func-req` will confirm and offer a UX-only path."*

**If N=0:**
> "Analysis complete. Sections 5-6 filled and Scope Proposal appended to `docs/projects/<name>/prd.md`.
>
> All stories have clear signals — no meeting needed. Review the Scope Proposal, confirm with PM, then run `/tech-design --project <name>`."

**If N>0:**
> "Analysis complete. Sections 5-6 filled and Scope Proposal appended to `docs/projects/<name>/prd.md`.
>
> Hold a joint meeting (~20 min) to resolve the ⚡ items in the Scope Proposal — each item has EA/GA cases and a recommendation to react to.
>
> After the meeting: move resolved ⚡ items to ✅ EA or ⏩ GA in the Scope Proposal, then run `/tech-design --project <name>`."

---

## Notes

- No status change needed after scope decisions. `/tech-design` and `/func-req` check `status: approved` on `prd.md` + no remaining ⚡ items in the Scope Proposal.
- Re-running `/analyze` is safe — it overwrites sections 5-6 and the Scope Proposal section if they already exist.
- Stories that depend on each other should be grouped and called out in the proposal.
- If GitHub MCP is unavailable, stop and ask the eng lead to configure it — don't proceed with manual input that degrades downstream quality.
