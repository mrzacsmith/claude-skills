# Commit

Structured git commits with emoji-prefixed messages.

## Usage

```
/commit                        # auto-detect changes, draft message
/commit fix login bug          # use provided message
/commit src/app.js             # stage specific files only
/commit -p                     # commit and push
/commit -p fix login bug       # commit with message and push
```

## Emoji Prefixes

| Prefix | Usage |
|--------|-------|
| 📦 NEW: | New features or additions |
| 👍 IMPROVE: | Improvements to existing features |
| 🐛 FIX: | Bug fixes |
| 🚀 READY: | Ready for release/production |
| 📖 DOCUMENT: | Documentation changes |
| 🤖 TESTING: | Test-related changes |
| ☣️ BREAKING: | Breaking changes |

## What It Does

1. Reads `git status`, `git diff`, and recent log to understand changes
2. Stages files (excludes secrets like `.env`)
3. Picks the right emoji prefix based on the change type
4. Commits with `Co-Authored-By: Claude` trailer
5. Pushes to current branch if `-p` flag is provided

## Install

```bash
cp commit.md ~/.claude/commands/
```
