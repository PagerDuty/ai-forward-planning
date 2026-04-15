---
name: prd
description: Use when a PM has a rough idea or draft document for a product feature and needs to turn it into a structured PRD with EA and GA milestones, interview-driven refinement, and optional AI-specific questions
---

# PRD Skill

Interview the PM to refine their input, then draft a structured PRD using the template. Supports pasted rough drafts, file references, or inline ideas.

## Arguments

```
/prd --project <name>
```

- `--project` — short slug for the product/feature (e.g., `alerting-v2`, `onboarding-flow`). Used as the folder name under `docs/projects/`.

## Steps

### Step 1: Collect input

Accept the PM's rough input in any of these forms:
- **Pasted draft** — PM pastes text from a doc, email, or notes
- **File reference** — PM provides a file path; read the file
- **Inline description** — PM types their idea directly

If no input is provided, ask: *"Share your rough draft or describe the feature — paste a doc, give a file path, or just describe it."*

Summarize what you understood in 2–3 sentences and ask the PM to confirm before proceeding to the interview.

---

### Step 2: Run the interview

Ask each question **one at a time**. Wait for the answer before asking the next. Do not ask all questions at once.

**General questions (always ask all 6):**

1. *"Who is the primary user for this feature? Describe their role and the context in which they'd use it."*

2. *"What does success look like 6 months after GA? Give 1–2 measurable outcomes you'd point to."*

3. *"For EA — who are the initial users (internal team, select customers, beta group?) and what's the rough rollout plan?"*

4. *"What are the hard must-haves for EA? What can wait for GA?"*

5. *"What are the explicit non-goals — things we are NOT building in this scope?"*

6. *"Any known dependencies (other teams, systems, APIs) or open questions that need answers before development starts?"*

**AI-specific questions (ask only if input mentions any of: AI, ML, model, LLM, prediction, classification, recommendation, neural, embedding, inference, fine-tun, training data, dataset, prompt, generative, GPT, Claude, OpenAI, Anthropic, vector, RAG, retrieval-augmented):**

7. *"Which model(s) are being considered, or is model selection part of the discovery work?"*

8. *"What data is available — training data, fine-tuning corpus, or context for retrieval? Any privacy or compliance constraints on that data?"*

9. *"How will AI output quality be measured — accuracy metrics, human review, A/B testing, user feedback loops?"*

10. *"What are the key failure modes and how should the system handle them? (e.g., hallucinations, low-confidence responses, model drift, latency spikes)"*

11. *"Are there responsible AI considerations — bias risks, explainability requirements, content guardrails, or audit trail needs?"*

---

### Step 3: Draft the PRD

Using the original input + all interview answers, fill in all sections of `template.md`. Be opinionated:
- Separate EA (must-haves, limited users) from GA (feature-complete, all edge cases)
- EA Known Gaps should list anything intentionally deferred
- Both milestones need a Misc / Launch Tasks section
- Success Metrics must be measurable — use the answer from question 2
- If AI-specific questions were asked, add an **AI Considerations** subsection under GA Requirements covering model selection, data, evaluation, failure modes, and responsible AI

---

### Step 4: Write file

```bash
mkdir -p docs/projects/<name>
```

Write to `docs/projects/<name>/prd.md` with frontmatter:
```yaml
---
project: <name>
milestone: both
created: <today's date YYYY-MM-DD>
status: draft
ai-project: true   # only include this line if AI questions were triggered
---
```

---

### Step 5: Prompt human review

> "PRD drafted and saved to `docs/projects/<name>/prd.md`. Please review and edit it. When you're happy with it, update `status: reviewed` in the frontmatter and let me know."

---

## Status Gates

Downstream skills check the `status` field:
- `draft` — not yet reviewed by human
- `reviewed` — human has read and edited it
- `approved` — explicitly signed off, safe to pass to JIRA

## Notes

- Follow the template structure exactly — downstream skills parse section headings
- EA and GA sections are both required even if GA is minimal
- Do not invent requirements — only use what came from the input and interview answers
- If the PM's rough draft already answers an interview question clearly, skip that question and note it was covered
