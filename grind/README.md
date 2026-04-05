# Grind

Work through open GitHub issues one at a time using TDD, feature branches, and PRs.

## Usage

```
/grind              # default: 5 issues, oldest first
/grind 8            # work through 8 issues
/grind 3 bug        # 3 issues with "bug" label
/grind #12 #15 #20  # specific issues
```

## Workflow Per Issue

1. **Read issue** — full body + comments, quick codebase scan
2. **Comment** — "Starting work on this."
3. **Branch** — `fix/<number>-slug`, `feat/<number>-slug`, or `chore/<number>-slug`
4. **TDD implementation** — red-green-refactor vertical slices, matching existing test patterns
5. **Commit + push** — emoji format, `Fixes #N` in final commit
6. **Create PR** — summary + test plan, `Fixes #N` for auto-close
7. **Comment on issue** — "PR ready: \<url\>"
8. **Return to default branch** — move to next issue

## After All Issues

- Detects dependency signals ("blocked by", "depends on", shared modules)
- Assigns merge order numbers
- Shows summary table with branch, PR, and recommended merge order

## Key Rules

- One issue at a time — finish or skip before moving on
- Feature branches only — never commit to default branch
- TDD is mandatory (except pure config/docs)
- Creates PRs but never merges them — that's your job
- Atomic branches — one issue per branch, no scope creep

## Requirements

- **GitHub CLI (`gh`)** — install with `brew install gh`, then `gh auth login`. See [cli.github.com](https://cli.github.com) for other platforms.

## Install

```bash
cp grind.md ~/.claude/commands/
```
