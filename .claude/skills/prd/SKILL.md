---
name: prd
description: Use when starting a new product feature or initiative and you have a rough idea to turn into a structured PRD with EA and GA milestones
---

# PRD Skill

Draft a structured PRD from a rough idea, save it for human review, and prepare it for downstream pipeline skills.

## Arguments

```
/prd --project <name>
```

- `--project` — short slug for the product/feature (e.g., `alerting-v2`, `onboarding-flow`). Used as the folder name under `docs/projects/`.

## Steps

1. **Collect the idea** — Ask the human for their rough idea if not provided inline. One open question: "What problem are you trying to solve and who has it?"

2. **Draft the PRD** — Fill in all sections of `template.md`. Be opinionated:
   - Separate EA (must-haves, limited users) from GA (feature-complete, all edge cases)
   - EA Known Gaps should list anything you're intentionally deferring
   - Both milestones need a Misc / Launch Tasks section
   - Success Metrics must be measurable, not vague

3. **Create project directory and write file**

   ```bash
   mkdir -p docs/projects/<name>
   ```

   Write the drafted PRD to `docs/projects/<name>/prd.md` with frontmatter:
   ```yaml
   ---
   project: <name>
   milestone: both
   created: <today's date YYYY-MM-DD>
   status: draft
   ---
   ```

4. **Prompt human review** — Say exactly:
   > "PRD drafted and saved to `docs/projects/<name>/prd.md`. Please review and edit it. When you're happy with it, update `status: reviewed` in the frontmatter and let me know."

## Status Gates

Downstream skills check the `status` field:
- `draft` — not yet reviewed by human
- `reviewed` — human has read and edited it
- `approved` — explicitly signed off, safe to pass to JIRA

## Notes

- Follow the template structure exactly — downstream skills parse section headings
- EA and GA sections are both required even if GA is minimal
- Do not invent requirements — only capture what the human described
