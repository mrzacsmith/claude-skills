# PRD to Plan

Turn a PRD into a phased implementation plan using tracer-bullet vertical slices, saved as a local Markdown file in `./plans/`.

## Step 0: Repo Guardrail

Before doing anything else, run:

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

- If this succeeds, store the `owner/repo` value for reference. Confirm with the user: "Working in **owner/repo**. Correct?"
- If the user says no, ask them to `cd` into the correct project directory and run `/prd-to-plan` again. Stop here.
- If the command fails, inform the user and stop.

---

## Step 1: Confirm the PRD is in Context

Ask the user for the PRD — either:
- A GitHub issue number (fetch with `gh issue view <number> --repo <owner/repo> --comments`)
- A local file path
- Or confirm it's already in the conversation from a previous `/write-prd` invocation

---

## Step 2: Explore the Codebase

Explore the codebase to understand:
- Project structure and tech stack
- Existing patterns and conventions
- How the PRD maps to current architecture

---

## Step 3: Identify Durable Architectural Decisions

Document decisions that will survive implementation changes:
- Route structures / URL patterns
- Database schema shape (Firestore collections, document structures)
- Key data models
- Authentication / authorization approach
- Third-party service boundaries (Firebase services, external APIs)

---

## Step 4: Draft Vertical Slices

Break the PRD into phased vertical slices. Each phase is a thin slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

### Rules
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Do NOT include specific file names, function names, or implementation details likely to change
- DO include durable decisions: route paths, schema shapes, data model names

---

## Step 5: Quiz the User

Present the proposed breakdown as a numbered list. For each phase show:
- **Title**
- **User stories covered** (from the PRD)

Ask:
- Does the granularity feel right? (too coarse / too fine)
- Should any phases be merged or split?
- Is the ordering correct?

**Iterate until the user approves. Do not write the plan file until approved.**

---

## Step 6: Write the Plan File

Create the `./plans/` directory if it doesn't exist.

Write a Markdown file named after the feature (e.g., `./plans/user-authentication.md`) using this template:

```markdown
# Plan: <Feature Name>

**PRD**: #<issue-number> (or link/reference)
**Repo**: <owner/repo>
**Created**: <date>

---

## Architectural Decisions

### Routes
- ...

### Schema
- ...

### Key Models
- ...

### Service Boundaries
- ...

---

## Phase 1: <Title>

**User stories**: 1, 2, 5

### What to build
<Description of end-to-end behavior for this slice>

### Acceptance criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

---

## Phase 2: <Title>

**User stories**: 3, 4

### What to build
<Description>

### Acceptance criteria
- [ ] ...

---

(Continue for all phases)
```

Display the file path to the user when done.

---

## Rules

1. **Always confirm the repo** before fetching any GitHub issues
2. **Use `--repo` flag** on all `gh` commands with the confirmed owner/repo
3. **Never write the plan** until the user approves the breakdown
4. **Vertical slices only** — no horizontal layer-by-layer phases
5. **Durable descriptions** — no file paths, function names, or implementation details that will change
