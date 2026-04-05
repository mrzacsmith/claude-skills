# PRD to Issues

Break a PRD into independently-grabbable GitHub issues using vertical slices (tracer bullets).

## Step 0: Repo Guardrail

Before doing anything else, run:

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

- If this succeeds, confirm the repo with the user: "Issues will be created in **owner/repo**. Correct?"
- If the user says no, ask them to `cd` into the correct project directory and run `/prd-to-issues` again. Stop here.
- If the command fails (no git repo, no GitHub remote, `gh` not authenticated), inform the user and stop. Do not proceed without a confirmed repo.

Store the `owner/repo` value — use it for all `gh` commands in this session.

---

## Step 1: Locate the PRD

Ask the user for the PRD GitHub issue number (or URL).

Fetch it with:

```bash
gh issue view <number> --repo <owner/repo> --comments
```

If the PRD is already in context from a previous `/write-prd` invocation, confirm with the user which PRD to use.

---

## Step 2: Explore the Codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code and how the PRD maps to existing architecture.

---

## Step 3: Draft Vertical Slices

Break the PRD into **tracer bullet** issues. Each issue is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices may be **HITL** or **AFK**:
- **HITL** (Human In The Loop): Requires human interaction, such as an architectural decision or a design review
- **AFK** (Away From Keyboard): Can be implemented and merged without human interaction

Prefer AFK over HITL where possible.

### Vertical Slice Rules
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones

---

## Step 4: Quiz the User

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **User stories covered**: which user stories from the PRD this addresses

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

**Iterate until the user approves the breakdown. Do not create issues until approved.**

---

## Step 5: Create the GitHub Issues

For each approved slice, create a GitHub issue using:

```bash
gh issue create --repo <owner/repo> --assignee @me --title "<slice title>" --body "$(cat <<'EOF'
<issue body here>
EOF
)"
```

**Create issues in dependency order** (blockers first) so you can reference real issue numbers in the "Blocked by" field.

Use the confirmed `owner/repo` from Step 0 on every `gh issue create` command.

After all issues are created, display a summary list of all issue numbers and URLs.

### Issue Template

```markdown
## Parent PRD

#<prd-issue-number>

## Type

HITL / AFK

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation. Reference specific sections of the parent PRD rather than duplicating content.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Blocked by

- Blocked by #<issue-number> (if any)

Or "None - can start immediately" if no blockers.

## User stories addressed

Reference by number from the parent PRD:

- User story 3
- User story 7
```

Do NOT close or modify the parent PRD issue.

---

## Rules

1. **Always confirm the repo** before any GitHub interaction
2. **Always use `--repo` flag** on all `gh` commands with the confirmed owner/repo
3. **Never create issues** until the user has approved the breakdown
4. **Create in dependency order** so blocker issue numbers are available for references
5. **Do not modify the parent PRD issue**
