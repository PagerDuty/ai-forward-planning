# PRD → JIRA Pipeline

A set of Claude Code skills that take a rough product idea from concept to JIRA tickets — with human review checkpoints at every stage.

## Pipeline

```
Rough idea → /prd → [review] → /tech-design (optional) → /func-req --milestone ea|ga → /jira-tickets --milestone ea|ga
```

## Skills

| Skill | Command | Purpose |
|-------|---------|---------|
| PRD | `/prd --project <name>` | Draft a PRD from a rough idea |
| Tech Design | `/tech-design --project <name>` | Generate deep technical spec from PRD + GitHub repos |
| Functional Requirements | `/func-req --project <name> --milestone ea\|ga` | Generate epic + stories scoped to a milestone |
| JIRA Tickets | `/jira-tickets --project <name> --milestone ea\|ga` | Create Epic + Stories in JIRA |

## Quick Start

1. Have a rough idea for a feature
2. Run `/prd --project my-feature`
3. Review and edit `docs/projects/my-feature/prd.md`, update `status: reviewed`
4. *(Optional)* Run `/tech-design --project my-feature` if the feature needs architecture work
5. Run `/func-req --project my-feature --milestone ea` to generate EA requirements
6. Run `/jira-tickets --project my-feature --milestone ea` to create EA JIRA tickets
7. Repeat steps 5–6 with `--milestone ga` for GA

## EA vs GA

| Milestone | Definition |
|-----------|-----------|
| **EA (Early Access)** | Core functionality, must-haves, limited users. Known gaps documented but deferred. |
| **GA (General Availability)** | Feature-complete, all edge cases covered, production-ready. |

Both milestones include a **Misc / Launch Tasks** bucket for rollout-specific work.

## Project Structure

```
docs/
  projects/
    <project-name>/
      prd.md                          # /prd output
      tech-design.md                  # /tech-design output (optional)
      functional-requirements-ea.md   # /func-req --milestone ea output
      functional-requirements-ga.md   # /func-req --milestone ga output
```

> Note: `/jira-tickets` creates tickets directly in JIRA and writes issue keys back into the functional-requirements file.

## Prerequisites

- [Claude Code](https://claude.ai/code) with skills support
- JIRA MCP configured and connected
- GitHub MCP configured (required only for `/tech-design`)
