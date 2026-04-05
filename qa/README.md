# QA (Quick Add)

Rapidly file one or more well-structured, AI-agent-friendly GitHub issues. Speed is the priority.

## Usage

```
/qa login page crashes on Safari
/qa login crashes; add dark mode; refactor nav
/qa                                              # interactive session mode
```

## Workflow

1. **Repo guardrail** — confirms the target repo
2. **Capture** — parses input into individual issues (splits on semicolons/newlines). If no args, enters interactive session mode.
3. **Quick clarification** — at most 2 questions total across all issues, only if truly ambiguous. Usually skipped.
4. **Codebase scan** — fast scan to identify relevant areas and pick up context for Agent Notes
5. **File issues** — creates in dependency order so "Blocked by" references resolve. Applies labels if they exist in the repo.
6. **Summary table** — shows all created issues with numbers, titles, types, and labels

## Issue Template Sections

- Type (Bug / Feature / Chore)
- Current Behavior / Expected Behavior
- Reproduction Steps
- Blocked By
- Scope (area + bounded definition of "done")
- Acceptance Criteria (testable, 2-4 per issue)
- Agent Notes (context for AI agents — no file paths, behaviors only)

## Key Rules

- Enrich, don't interrogate — flesh out rough input using judgment and codebase context
- Every issue is independently actionable
- No file paths or implementation details in issue bodies (they go stale)
- Files dependencies in order

## Requirements

- `gh` CLI authenticated with access to the target repo

## Install

```bash
cp qa.md ~/.claude/commands/
```
