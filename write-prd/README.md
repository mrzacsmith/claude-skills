# Write PRD

Create a Product Requirements Document through an interactive interview process, then file it as a GitHub issue.

## Usage

```
/write-prd
```

## Workflow

1. **Repo guardrail** — confirms the target GitHub repo before doing anything
2. **Problem description** — asks for a detailed description of the problem and potential solutions
3. **Codebase exploration** — scans the repo to verify assertions and understand current state
4. **Interview** — relentlessly questions every aspect of the plan until shared understanding is reached
5. **Module design** — sketches deep modules with simple, testable interfaces; gets approval on which to test
6. **Submit PRD** — writes the PRD using the template and files it as a GitHub issue

## PRD Template Sections

- Problem Statement
- Solution
- User Stories (extensive numbered list)
- Implementation Decisions (no file paths — durable descriptions only)
- Testing Decisions
- Out of Scope
- Further Notes

## Requirements

- `gh` CLI authenticated with access to the target repo

## Install

```bash
cp write-prd.md ~/.claude/commands/
```
