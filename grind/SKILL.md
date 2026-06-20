---
name: grind
description: Work through open GitHub issues one at a time, each executed as a complete unit of work through a feedback loop (TDD then validate, never advancing on red) behind an enforced lint-staged and Prettier quality gate, then shipped as a feature branch and PR. Use to grind issues or when the user runs /grind.
---

# Grind Issues

Work through open GitHub issues one at a time — each issue executed as a complete
**unit of work** through a feedback loop (TDD → validate → never advance on red),
behind an enforced quality gate (lint-staged + Prettier + precommit hook), then
shipped as a feature branch + PR.

Grind is the **orchestrator**. It composes two other skills rather than
reimplementing them:

- **`/code-quality`** — sets up the quality gate once per repo (husky +
  lint-staged + Prettier + CI lint gate). Grind ensures it exists, then relies on
  it.
- **`/do-work`** — executes each issue through the feedback loop (build →
  validate → commit, never advancing on a red signal). Grind hands each issue to
  it when available.

Grind owns the **queue**: fetching issues, branching, ordering by dependency,
opening PRs, and reporting.

## Arguments

- Number of issues to work through (default: 5): `/grind 8`
- Optional label filter: `/grind 8 bug` or `/grind 3 enhancement`
- Optional issue numbers: `/grind #12 #15 #20`

---

## Step 0: Repo Setup

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

- If this fails, inform the user and stop.
- Store the `owner/repo` value for all `gh` commands.
- Identify the default branch:
  ```bash
  gh repo view --json defaultBranchRef -q .defaultBranchRef.name
  ```

### Quality gate (feedback-loop enforcement)

Grind depends on a quality gate so every commit is auto-formatted and
lint-checked — that's what makes the feedback loop robust instead of optional.
Detect whether it exists:

```bash
ls .husky/pre-commit 2>/dev/null && echo "precommit hook ✓"
grep -q '"lint-staged"' package.json 2>/dev/null && echo "lint-staged ✓"
grep -qE '"(lint|typecheck|build)"[[:space:]]*:' package.json 2>/dev/null && echo "scripts ✓"
```

- **If the gate is missing**, offer to run `/code-quality` once before grinding —
  it installs husky + lint-staged + Prettier + a CI lint gate idempotently. The
  CI gate fires on the PRs grind opens, so it's the real team-wide backstop (the
  local hook is fast but bypassable).
- **If the user declines**, continue anyway — grind degrades gracefully to
  whatever checks exist. A missing hook is never a hard blocker.
- **If present**, note the project's real validation commands (`lint`,
  `typecheck`, `build`, test runner). The issue loop (Step 2c) runs these.

---

## Step 1: Fetch Issues

Pull the issue queue:

```bash
gh issue list --repo <owner/repo> --state open --assignee @me --limit 50 --json number,title,labels,body
```

- If the user specified issue numbers, fetch only those.
- If the user specified a label filter, add `--label <label>`.
- Sort by issue number (oldest first) unless the user says otherwise.
- Display the queue to the user:

```
Found N open issues. Working on the first M:

| # | Title | Labels |
|---|-------|--------|
| #12 | Fix login timeout | bug |
| #15 | Add CSV export | enhancement |
```

- **Wait for user confirmation** before starting: "Look good? I'll start with #12."

---

## Step 2: Issue Loop

For each issue in the queue (up to the requested count):

### 2a. Understand the Issue

- Read the full issue body and comments.
- Do a quick codebase scan to understand the relevant area — identify test files, modules, and patterns involved.
- Comment on the issue (concise):
  ```bash
  gh issue comment <number> --repo <owner/repo> --body "Starting work on this."
  ```

### 2b. Create Feature Branch

Branch from the default branch. Use this naming convention:

- Bug fixes: `fix/<number>-brief-slug`
- Features: `feat/<number>-brief-slug`
- Chores: `chore/<number>-brief-slug`

```bash
git checkout <default-branch>
git pull origin <default-branch>
git checkout -b fix/<number>-brief-slug
```

Keep the slug to 3-4 words max, lowercase, hyphenated.

### 2c. Execute the Issue (feedback loop)

Each issue is a complete **unit of work**: build it, prove it, commit it — never
advancing on a red signal. **If `/do-work` is available, hand the issue to it** and
let it drive the loop. Otherwise run the loop inline:

**1. TDD — the behavioral loop** (red → green → refactor, vertical slices):

1. **Identify behaviors** — From the issue, determine what testable behaviors need to exist.
2. **Write one test** — Write a test for the first behavior. Run it. Confirm it fails (RED).
3. **Implement** — Write minimal code to make it pass (GREEN).
4. **Repeat** — Next behavior, next test, next implementation. One at a time.
5. **Refactor** — Once all tests pass, clean up. Run tests after each refactor step.

Rules:
- One test at a time — vertical slices, never horizontal.
- Tests verify behavior through public interfaces, not implementation.
- Never refactor while RED.
- Mock only at system boundaries (external APIs, time, etc.).
- If the project has existing test patterns, follow them.
- If no test infrastructure exists, set it up minimally before starting.

**2. Validate — the correctness loop.** Before committing, run the project's real
signals (from Step 0), cheapest first, and **loop until all green**:

```
typecheck / lint  →  tests  →  build
```

On any red: fix the cause, re-run **that** check, repeat. **Do not commit while
any signal is red.** This loop is the fast feedback; the precommit hook (next
step) is the safety net on top of it — not a substitute for it.

**3. Self-review.** Read the diff as if reviewing someone else's PR: does it do
exactly what the issue asked (no scope creep), match conventions, leave no debug
cruft? If not, loop back to step 1.

### 2d. Commit and Push

Stage and commit with a message that references the issue:

```bash
git add <specific-files>
git commit -m "$(cat <<'EOF'
<emoji> <TYPE>: <concise description>

Fixes #<number>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
git push -u origin <branch-name>
```

**The precommit hook runs here.** When the quality gate is present, lint-staged
auto-formats the staged files with Prettier and blocks the commit on lint errors —
so the committed tree may differ from what you staged. That's expected; it's the
enforcement layer. **Never bypass it with `--no-verify`** to force a red commit
through; fix the cause and re-commit.

Use the emoji commit format from `/commit`:
- `🐛 FIX:` for bug fixes
- `📦 NEW:` for new features
- `👍 IMPROVE:` for improvements
- `🤖 TESTING:` for test-only changes

If the work requires multiple commits (e.g., test infra setup, then implementation), make atomic commits along the way. Only the final commit needs `Fixes #N`.

### 2e. Create PR

```bash
gh pr create --repo <owner/repo> --title "<emoji> <TYPE>: <concise title>" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points of what changed and why>

Fixes #<number>

## Test Plan
- [ ] <what was tested>
- [ ] All existing tests still pass

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

The `Fixes #<number>` in the PR body ensures the issue auto-closes when the PR is merged.

### 2f. Update the Issue

Add a concise comment linking the PR:

```bash
gh issue comment <number> --repo <owner/repo> --body "PR ready: <pr-url>"
```

### 2g. Move to Next

```bash
git checkout <default-branch>
```

Report progress to the user:

```
Completed #12 (Fix login timeout) -> PR #25
Moving to #15 (Add CSV export)... (3 of 8 remaining)
```

---

## Step 3: Dependency Detection

Before displaying the summary, scan issue bodies and PR descriptions for dependency signals:
- "blocked by #N", "depends on #N", "after #N", "requires #N"
- Issues that touch the same files/modules (detected during implementation)

Assign a merge order number:
- Independent issues all get order `1` (merge in any order).
- If issue B depends on issue A, A gets a lower order number than B.
- If a chain exists (A -> B -> C), number them sequentially.

---

## Step 4: Summary

After completing all issues (or if stopped early), display:

```
Grind complete. Worked on N issues:

| Merge | Issue | Title | Branch | PR | Status |
|-------|-------|-------|--------|----|--------|
| 1 | #12 | Fix login timeout | fix/12-login-timeout | #25 | PR ready |
| 1 | #18 | Refactor auth | chore/18-refactor-auth | #27 | PR ready |
| 2 | #15 | Add CSV export (depends on #12) | feat/15-csv-export | #26 | PR ready — merge after #25 |
```

**Merge column**: PRs with the same number can be merged in any order. Lower numbers must be merged first. After merging a PR, rebase the next one onto updated main before merging.

If all issues are independent, omit the merge column and note: "All PRs are independent — merge in any order."

Return to the default branch when done.

---

## Rules

1. **One issue at a time** — finish or explicitly skip before moving on
2. **Always use feature branches** — never commit directly to the default branch
3. **Quality gate first** — ensure the lint-staged/Prettier precommit gate exists (offer `/code-quality` if missing); rely on it, never bypass it
4. **TDD is mandatory** — write tests first, then implement. Skip only if the issue is purely config/docs
5. **Never commit on red** — typecheck, lint, tests, and build must be green before the commit; no `--no-verify` to force a failing commit through
6. **Commits reference issues** — use `Fixes #N` in the final commit and PR body
7. **Comments are concise** — "Starting work." and "PR ready: <url>" are enough. No essays.
8. **Don't merge PRs** — create them and leave for the user to review and merge
9. **Ask before skipping** — if an issue is unclear or blocked, ask the user rather than silently skipping
10. **Respect existing patterns** — match the project's test framework, code style, and conventions
11. **Atomic branches** — each branch addresses exactly one issue. No scope creep.
12. **Stay on the default branch between issues** — always return before starting the next one
