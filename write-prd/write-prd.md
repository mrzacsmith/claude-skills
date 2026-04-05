# Write a PRD

Create a Product Requirements Document through an interactive interview process, then submit it as a GitHub issue in the current repository.

## Step 0: Repo Guardrail

Before doing anything else, run:

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

- If this succeeds, confirm the repo with the user: "This PRD will be filed as a GitHub issue in **owner/repo**. Correct?"
- If the user says no, ask them to `cd` into the correct project directory and run `/write-prd` again. Stop here.
- If the command fails (no git repo, no GitHub remote, `gh` not authenticated), inform the user and stop. Do not proceed without a confirmed repo.

Store the `owner/repo` value — use it for all `gh` commands in this session.

---

## Step 1: Problem Description

Ask the user for a long, detailed description of the problem they want to solve and any potential ideas for solutions.

**Wait for their response before proceeding.**

---

## Step 2: Codebase Exploration

Explore the repo to verify the user's assertions and understand the current state of the codebase. Look at project structure, relevant modules, existing patterns, and anything that relates to the problem described.

---

## Step 3: Interview

Interview the user relentlessly about every aspect of this plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one.

Keep asking questions until there are no ambiguities left. Do not rush this step.

**Wait for the user's answers before proceeding.**

---

## Step 4: Module Design

Sketch out the major modules you will need to build or modify to complete the implementation. Actively look for opportunities to extract deep modules that can be tested in isolation.

A deep module (as opposed to a shallow module) is one which encapsulates a lot of functionality in a simple, testable interface which rarely changes.

Present the modules to the user and ask:
- Do these modules match your expectations?
- Which modules do you want tests written for?

**Wait for approval before proceeding.**

---

## Step 5: Write and Submit the PRD

Once you have a complete understanding of the problem and solution, write the PRD using the template below and submit it as a GitHub issue using:

```bash
gh issue create --repo <owner/repo> --title "PRD: <feature name>" --body "$(cat <<'EOF'
<prd content here>
EOF
)"
```

Use the confirmed `owner/repo` from Step 0.

After creation, display the issue URL to the user.

### PRD Template

```markdown
## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

This list should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this PRD.

## Further Notes

Any further notes about the feature.
```

---

## Rules

1. **Always confirm the repo** before any GitHub interaction
2. **Always wait for user responses** at each interview step — never skip ahead
3. **Never assume** — if something is unclear, ask
4. **Use `--repo` flag** on all `gh` commands with the confirmed owner/repo
