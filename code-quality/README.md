# Code Quality

Idempotently bring a JavaScript repo up to the standard **Code Quality Playbook** baseline (Prettier, ESLint ratchet, husky + lint-staged, CI lint gate, optional JSDoc typecheck), then optionally migrate it from **npm → pnpm**. Present pieces are skipped; missing ones are added — so it's safe to run on any JS repo, repeatedly.

## Usage

```
/code-quality
```

Run it inside a JavaScript repo. It detects what's already in place, adds only what's missing, and asks before the optional pnpm migration.

## How It Works

- Works through Steps 1–4 of the **Code Quality Playbook** (`references/playbook.md`), detecting each piece and adding only what's missing
- Optional Step 5: migrate npm → pnpm (`references/pnpm-migration.md`) — lockfile import, `packageManager` pin, build-script allow-list, Firebase functions-framework fix, npm cleanup
- Manual and global: runs only when you invoke `/code-quality`, and works on **any** JS repo
- Idempotent — re-running never duplicates config

## When to Use

- Standing up linting/formatting/CI gates on a repo that lacks them
- Bringing an inherited or inconsistent JS repo up to a shared baseline
- Migrating a repo from npm to pnpm with the Firebase/build-script gotchas handled

## Install

```bash
cp -R code-quality ~/.claude/skills/
```
