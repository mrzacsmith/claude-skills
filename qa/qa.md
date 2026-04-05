# Quick Add Issues

Rapidly file one or more well-structured, AI-agent-friendly GitHub issues in the current repo.

## Step 0: Repo Guardrail

Before doing anything else, run:

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

- If this succeeds, confirm the repo with the user: "Filing issues in **owner/repo**. Correct?"
- If the user says no, ask them to `cd` into the correct project directory and run `/qa` again. Stop here.
- If the command fails (no git repo, no GitHub remote, `gh` not authenticated), inform the user and stop.

Store the `owner/repo` value — use it for all `gh` commands in this session.

---

## Step 1: Capture Input

The user provides one or more issue descriptions as arguments. These can be:

- A single one-liner: `/qa login page crashes on Safari`
- Multiple issues separated by newlines or semicolons: `/qa login crashes on Safari; add dark mode toggle; refactor nav component`
- A paragraph describing several things they noticed

**Parse the input into individual issues.** If a single description clearly contains multiple distinct items, split them. If ambiguous, ask once: "I see X items here — does that look right?"

If no arguments were provided, ask: "What issues do you want to file? You can list multiple separated by semicolons, or we can do an interactive QA session — just say 'session'."

**Interactive session mode**: If the user says "session" or provides no batch list, enter a conversational loop: take one issue at a time, file it, then ask "What else?" Repeat until the user says they're done. Each issue is independent — don't carry context between them.

---

## Step 2: Quick Clarification (Optional)

For each issue, determine if the one-liner is clear enough to file immediately. In most cases it will be.

Only ask clarifying questions if:
- You genuinely cannot determine bug vs feature vs chore
- The description is too vague to write a testable acceptance criterion (e.g., "fix the thing")

Ask at most **2 questions total** across all issues, not per issue. Batch them into a single message. If everything is clear, skip this step entirely.

---

## Step 3: Quick Codebase Scan

Do a fast scan of the codebase — not deep exploration, just enough to:
- Identify which area/module each issue relates to
- Verify the descriptions make sense in context
- Pick up any relevant details for the Agent Notes section (gotchas, related patterns, ongoing work)

Keep this under 30 seconds. Speed is the priority.

---

## Step 4: File All Issues

Create each issue using `gh issue create --repo <owner/repo>`. Apply labels if the repo has them (check with `gh label list --repo <owner/repo> --limit 50` once before creating).

**Ordering and dependencies**: When issues have dependencies (e.g., one fix requires another to land first), file them in dependency order so that "Blocked by" references resolve to real issue numbers. After each creation, capture the issue number and URL.

Use this template for each issue body:

```markdown
## Type

Bug / Feature / Chore

## Current Behavior

What happens now. For new features, state "N/A — new capability."

## Expected Behavior

What should happen — written as a testable assertion. Be specific enough that an AI agent or developer can determine when this is satisfied.

## Reproduction Steps

1. Step 1
2. Step 2

For features or chores: "N/A"

## Blocked By

If this issue depends on another issue being completed first, list the issue number(s) here (e.g., #123). Otherwise, state "None."

## Scope

- **Area**: e.g., "Authentication", "Dashboard", "Firestore rules", "Navigation" — describe by behavior/domain, not file paths
- **Bounded**: Yes — [brief statement of what "done" looks like]

## Acceptance Criteria

- [ ] Specific, testable criterion
- [ ] Another criterion
- [ ] (aim for 2-4 crisp criteria per issue)

## Agent Notes

Context an AI agent would need that isn't obvious from the code:
- Describe relevant behaviors, areas, and patterns — do NOT reference specific file paths or line numbers (these go stale after refactors)
- Things to NOT touch or depend on
- Known gotchas or ongoing work in this area
- If none, state "No special considerations."
```

For the `gh issue create` command:

```bash
gh issue create --repo <owner/repo> --assignee @me --title "<concise title>" --label "<label>" --body "$(cat <<'EOF'
<issue body>
EOF
)"
```

If labels don't exist in the repo, omit the `--label` flag rather than creating new labels.

---

## Step 5: Summary

After all issues are created, display a summary table:

```
Created X issues in owner/repo:

| # | Title | Type | Labels |
|---|-------|------|--------|
| #123 | Login crashes on Safari | Bug | bug |
| #124 | Add dark mode toggle | Feature | enhancement |
| #125 | Refactor nav component | Chore | — |
```

---

## Rules

1. **Speed is the priority** — minimize questions, maximize filing speed
2. **Always confirm the repo** before any GitHub interaction
3. **Use `--repo` flag** on all `gh` commands with the confirmed owner/repo
4. **Enrich, don't interrogate** — take rough input and flesh it out into the structured template using your own judgment and codebase context
5. **Every issue must be independently actionable** — an agent or developer should be able to pick it up with no additional context needed
6. **Acceptance criteria must be testable** — not vague ("improve performance") but specific ("page loads in under 2 seconds")
7. **Skip clarification** if the input is clear enough to file — err on the side of filing fast
8. **Issues must be durable** — never include file paths, line numbers, or internal implementation details in issue bodies. Describe behaviors and domain areas instead. These references go stale after refactors.
9. **File dependencies in order** — when breaking down into multiple issues, create them in dependency order so "Blocked by #N" references point to real, already-created issues. Prefer many thin parallel issues over few thick ones; mark blocking relationships honestly.
