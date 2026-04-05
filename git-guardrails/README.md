# Git Guardrails

Set up a Claude Code PreToolUse hook that blocks dangerous git commands before they execute.

## Usage

```
/git-guardrails
```

## What Gets Blocked

| Command | Why |
|---------|-----|
| `git push` (all variants) | Prevents accidental pushes |
| `git reset --hard` | Prevents loss of uncommitted work |
| `git clean -f` / `-fd` | Prevents deletion of untracked files |
| `git branch -D` | Prevents force-deletion of branches |
| `git checkout .` / `git restore .` | Prevents bulk discard of changes |

## Setup Steps

1. **Ask scope** — project-only (`.claude/settings.json`) or global (`~/.claude/settings.json`)
2. **Create hook script** — `block-dangerous-git.sh` at the chosen location
3. **Add to settings** — merges the PreToolUse hook config into existing settings
4. **Customize** — asks if you want to add/remove patterns
5. **Verify** — runs a test to confirm blocking works

## Files

- `git-guardrails.md` — the skill definition
- `block-dangerous-git.sh` — the hook script (included as reference; the skill creates it at the correct path during setup)

## Requirements

- **`jq`** — install with `brew install jq` (macOS), `sudo apt install jq` (Linux), or see [jqlang.github.io/jq](https://jqlang.github.io/jq/download/)

## Install

```bash
cp git-guardrails.md ~/.claude/commands/
```
