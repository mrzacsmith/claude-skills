# Catchup

Read all uncommitted git changes into the conversation so Claude has full context on what you're working on.

## Usage

```
/catchup
```

## What It Does

1. Runs `git status`, `git diff`, `git diff --cached`, and `git log` in parallel
2. Summarizes all modified, added, and deleted files with descriptions from the diff
3. Notes which changes are staged vs unstaged
4. Asks if you want any files read in full for deeper context

## Output

- **Branch**: current branch name
- **Staged changes**: list of files
- **Unstaged changes**: list of files
- **Summary**: what's being worked on

This is read-only — no files are modified.

## Install

```bash
cp catchup.md ~/.claude/commands/
```
