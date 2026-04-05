# PRD to Plan

Convert a PRD into a phased implementation plan using tracer-bullet vertical slices. Saves the plan as Markdown in `./plans/`.

## Usage

```
/prd-to-plan
```

Accepts the PRD as a GitHub issue number, local file path, or from a previous `/write-prd` in the same conversation.

## Workflow

1. **Repo guardrail** — confirms the target repo
2. **PRD confirmation** — fetches or confirms the PRD is in context
3. **Codebase exploration** — maps PRD to current architecture
4. **Architectural decisions** — documents durable decisions (routes, schema, models, service boundaries)
5. **Vertical slices** — breaks the PRD into thin end-to-end phases, each demoable on its own
6. **User quiz** — iterates on granularity, ordering, and merge/split until approved
7. **Write plan file** — saves to `./plans/<feature-name>.md`

## Plan Template Sections

- Architectural Decisions (routes, schema, key models, service boundaries)
- Phases with user stories covered, what to build, and acceptance criteria

## Key Rules

- Vertical slices only — no horizontal layer-by-layer phases
- Durable descriptions — no file paths or function names that will change
- Never writes the plan until you approve the breakdown

## Requirements

- **GitHub CLI (`gh`)** — install with `brew install gh`, then `gh auth login`. See [cli.github.com](https://cli.github.com) for other platforms. Required when fetching a PRD from GitHub issues.

## Install

```bash
cp prd-to-plan.md ~/.claude/commands/
```
