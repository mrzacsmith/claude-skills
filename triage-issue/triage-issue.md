# Triage Issue

Investigate a reported problem, find its root cause, and create a GitHub issue with a TDD fix plan. This is a mostly hands-off workflow — minimize questions to the user.

## Step 0: Repo Guardrail

Before doing anything else, run:

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

- If this succeeds, store the `owner/repo` value. Confirm with the user: "Filing issue in **owner/repo**. Correct?"
- If the user says no, ask them to `cd` into the correct project directory and run `/triage-issue` again. Stop here.
- If the command fails (no git repo, no GitHub remote, `gh` not authenticated), inform the user and stop.

Use the confirmed `owner/repo` for all `gh` commands.

---

## Step 1: Capture the Problem

Get a brief description of the issue from the user. If they haven't provided one, ask ONE question: "What's the problem you're seeing?"

Do NOT ask follow-up questions yet. Start investigating immediately.

---

## Step 2: Explore and Diagnose

Deeply investigate the codebase. Your goal is to find:

- **Where** the bug manifests (entry points, UI, API responses)
- **What** code path is involved (trace the flow)
- **Why** it fails (the root cause, not just the symptom)
- **What** related code exists (similar patterns, tests, adjacent modules)

Look at:
- Related source files and their dependencies
- Existing tests (what's tested, what's missing)
- Recent changes to affected files (`git log` on relevant files)
- Error handling in the code path
- Similar patterns elsewhere in the codebase that work correctly

---

## Step 3: Identify the Fix Approach

Based on your investigation, determine:

- The minimal change needed to fix the root cause
- Which modules/interfaces are affected
- What behaviors need to be verified via tests
- Whether this is a regression, missing feature, or design flaw

---

## Step 4: Design TDD Fix Plan

Create a concrete, ordered list of RED-GREEN cycles. Each cycle is one vertical slice:

- **RED**: Describe a specific test that captures the broken/missing behavior
- **GREEN**: Describe the minimal code change to make that test pass

### Rules
- Tests verify behavior through public interfaces, not implementation details
- One test at a time, vertical slices (NOT all tests first, then all code)
- Each test should survive internal refactors
- Include a final refactor step if needed
- **Durability**: Describe behaviors and contracts, not internal structure. Tests assert on observable outcomes (API responses, UI state, user-visible effects), not internal state

---

## Step 5: Create the GitHub Issue

Create a GitHub issue using:

```bash
gh issue create --repo <owner/repo> --title "<concise issue title>" --body "$(cat <<'EOF'
<issue body>
EOF
)"
```

Use this template for the body:

```markdown
## Problem

A clear description of the bug or issue, including:
- What happens (actual behavior)
- What should happen (expected behavior)
- How to reproduce (if applicable)

## Root Cause Analysis

Describe what was found during investigation:
- The code path involved
- Why the current code fails
- Any contributing factors

Do NOT include specific file paths, line numbers, or implementation details that couple to current code layout. Describe modules, behaviors, and contracts instead.

## TDD Fix Plan

1. **RED**: Write a test that [describes expected behavior]
   **GREEN**: [Minimal change to make it pass]

2. **RED**: Write a test that [describes next behavior]
   **GREEN**: [Minimal change to make it pass]

...

**REFACTOR**: [Any cleanup needed after all tests pass]

## Acceptance Criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] All new tests pass
- [ ] Existing tests still pass
```

After creating the issue, display the issue URL and a one-line summary of the root cause.

---

## Rules

1. **Always confirm the repo** before any GitHub interaction
2. **Use `--repo` flag** on all `gh` commands
3. **Minimize questions** — investigate first, ask only if truly stuck
4. **Durable descriptions** — no file paths or line numbers in the issue body
5. **Create the issue directly** — don't ask the user to review before filing
