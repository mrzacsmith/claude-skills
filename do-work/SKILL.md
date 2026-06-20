---
name: do-work
description: >
  Execute one complete unit of work end-to-end through tight feedback loops —
  understand it, plan it, build it, validate it, commit it — never advancing a
  phase until its feedback signal is green. Manual and global: runs only when the
  user invokes /do-work or another skill explicitly hands it a scoped unit of
  work. Use to "do the work", "execute this task/plan/issue", "build and validate
  and commit", or as the execution engine other planning skills delegate to.
disable-model-invocation: true
allowed-tools: Bash Read Write Edit Grep Glob Task TodoWrite AskUserQuestion
---


# Do Work

Execute a complete unit of work: plan it, build it, validate it, commit it —
driven by **feedback loops**. Each phase produces output that you check against a
real signal (the user, the type checker, the tests, the build, your own review),
and you **loop until that signal is green before moving on**. Work never advances
on a red signal.

**Manual and global.** It runs only when:

- the user invokes `/do-work [task, issue #, or plan reference]`, or
- another skill explicitly hands off a scoped unit of work to it (e.g.
  `/prd-to-plan` or `/grind` says "now run `/do-work` on slice 1").

It is the **execution engine** — the counterpart to the planning skills. Planning
skills decide *what* to build; `do-work` builds it, proves it works, and commits.

---

## The core idea — feedback loops

A **unit of work** is the smallest slice that can be planned, built, validated,
and committed as one coherent change. You drive it through a chain of loops. Each
loop **acts, reads a feedback signal, and repeats until that signal is green**
before the next loop begins:

| Loop | Action | Feedback signal | Advance when |
| --- | --- | --- | --- |
| **Understand** | read the task, explore the code | your restatement matches the user's intent | scope is unambiguous |
| **Plan** (optional) | break the unit into steps | the plan covers the whole unit, no gaps | steps are concrete |
| **Implement** | write the smallest increment | it compiles / runs locally | increment is in place |
| **Validate** | run typecheck → lint → tests → build | all green | **zero red signals** |
| **Self-review** | read your own diff | it does what the task asked, nothing extra | you'd approve this PR |
| **Commit** | stage + commit (+ PR if asked) | commit succeeds, hooks pass | change is captured |

**Two rules make the loops work:**

1. **Cheapest signal first.** Inside the Validate loop, run the fastest check that
   can fail first (typecheck/lint, ~seconds) before the slow ones (full test
   suite, build, runtime). A fast red signal saves a slow one.
2. **Never advance on red.** A failing signal feeds straight back into the next
   Implement iteration — fix the cause, re-run the *same* check, and only move
   outward once it's green. Don't accumulate red signals to fix "later."

If a loop won't converge after a few honest iterations (test keeps failing for a
new reason each time, scope keeps growing), **stop and surface it** rather than
thrashing — report what's green, what's red, and what you think is wrong.

---

## Workflow

### 1. Understand the task

Read any referenced plan, PRD, or issue. Explore the codebase to understand the
relevant files, patterns, and conventions — match what's there, don't invent a
new style. **Feedback loop:** restate the unit of work in one or two sentences. If
the task is ambiguous or larger than one coherent slice, ask the user to clarify
or split scope **before** proceeding. Don't start building on a guess.

### 2. Plan the implementation (optional)

If the task has not already been planned, create a short plan for it — the ordered
steps to take it from here to committed. Use `TodoWrite` to track them so progress
is visible. If a plan already exists (a PRD, a `/prd-to-plan` output, an issue
with a checklist), use it; don't re-plan.

### 3. Implement

Work through the plan **one small increment at a time**. After each increment,
keep it in a runnable state — this is what makes the Validate loop fast and the
failures legible. Don't write the whole feature, then discover at the end that
none of it compiles.

### 4. Validate — the central loop

Run the project's real signals, cheapest first, and **loop until all green**:

```
typecheck / lint  →  unit tests  →  build  →  run the actual behavior
```

- Use the project's own commands (read `package.json` scripts / `Makefile` /
  CI config — don't assume). Examples: `npm run lint`, `npm run typecheck`,
  `npm test`, `npm run build`.
- On **any** red: stop, read the error, fix the cause, re-run **that** check.
  Repeat until green, then move to the next signal.
- For behavior changes, prove it actually works — run the thing, hit the
  endpoint, exercise the path. Tests passing is necessary, not always sufficient.
- If there's a `/verify`, `/run`, or `/tdd` skill in scope, use it here.

Do not proceed to commit while any signal is red.

### 5. Self-review

Read your own diff end to end as if reviewing someone else's PR:

- Does it do **exactly** what the task asked — no more (no scope creep), no less?
- Any debug logging, commented-out code, or stray files left behind?
- Does it match the surrounding conventions?

If review surfaces a problem, **loop back** to Implement → Validate. Only continue
when you'd approve this change.

### 6. Commit

Branch first if on the default branch (`main`/`master`). Stage and commit using
the project's convention (use `/commit` if available). Open a PR **only if the
user or the delegating skill asked for one**. Let the pre-commit hooks run — if
they fail, that's another feedback signal: fix and re-commit.

---

## Operating rules

- **One unit at a time.** If the task is really several units, do the first one
  fully (through commit) before the next, or ask the user how to split it.
- **Use the project's signals, not generic ones.** Discover the real lint / test /
  build / typecheck commands from the repo before running anything.
- **Don't fake green.** Never delete a failing test, loosen an assertion, or
  `--no-verify` past a hook to make a signal pass. A green you engineered is a red
  you hid.
- **Don't commit or push unless asked** (beyond the working commit that captures
  the unit). PRs, force-pushes, deploys → only on explicit request.
- **Surface blockers fast.** A loop that won't converge is information — report it
  instead of silently thrashing.

## Done when

The unit of work is understood, built, **every validation signal is green**, the
diff has been self-reviewed, and the change is committed — with nothing faked to
get there and nothing pushed/deployed that the user didn't ask for.
