# PRD to Issues

Break a PRD into independently-grabbable GitHub issues using vertical slices (tracer bullets).

## Usage

```
/prd-to-issues
```

Provide the PRD as a GitHub issue number or confirm it's already in context.

## Workflow

1. **Repo guardrail** — confirms the target repo
2. **Locate PRD** — fetches from GitHub or confirms from conversation
3. **Codebase exploration** — maps PRD to existing architecture
4. **Draft vertical slices** — thin end-to-end slices, each demoable on its own
5. **Quiz** — iterates on granularity, dependencies, HITL/AFK classification until approved
6. **Create issues** — files in dependency order so "Blocked by" references resolve

## Issue Types

- **AFK** (Away From Keyboard) — can be implemented and merged without human interaction
- **HITL** (Human In The Loop) — requires a decision, review, or design input

AFK is preferred where possible.

## Issue Template Sections

- Parent PRD (reference to original issue)
- Type (HITL / AFK)
- What to build (end-to-end behavior, not layer-by-layer)
- Acceptance criteria
- Blocked by
- User stories addressed (by number from PRD)

## Key Rules

- Never creates issues until you approve the breakdown
- Creates in dependency order
- Does not modify the parent PRD issue
- Vertical slices only — each slice cuts through all layers

## Requirements

- **GitHub CLI (`gh`)** — install with `brew install gh`, then `gh auth login`. See [cli.github.com](https://cli.github.com) for other platforms.

## Install

```bash
cp prd-to-issues.md ~/.claude/commands/
```
