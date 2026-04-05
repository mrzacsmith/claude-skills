# Triage Issue

Investigate a reported problem, find root cause, and file a GitHub issue with a TDD fix plan. Mostly hands-off — minimal questions, maximum investigation.

## Usage

```
/triage-issue
```

## Workflow

1. **Repo guardrail** — confirms the target repo
2. **Capture** — asks one question if needed: "What's the problem you're seeing?"
3. **Explore and diagnose** — deep codebase investigation: traces code paths, checks git history, finds root cause
4. **Fix approach** — determines minimal change, affected modules, classification (regression/missing feature/design flaw)
5. **TDD fix plan** — ordered RED-GREEN cycles as vertical slices
6. **Create GitHub issue** — files directly without asking for review

## Issue Template Sections

- Problem (actual vs expected behavior, reproduction steps)
- Root Cause Analysis (code path, why it fails — no file paths or line numbers)
- TDD Fix Plan (numbered RED/GREEN cycles + refactor step)
- Acceptance Criteria

## Key Rules

- Investigates first, asks questions only if truly stuck
- Durable descriptions — no file paths or line numbers in the issue body
- Files the issue directly — no review step

## Requirements

- `gh` CLI authenticated with access to the target repo

## Install

```bash
cp triage-issue.md ~/.claude/commands/
```
