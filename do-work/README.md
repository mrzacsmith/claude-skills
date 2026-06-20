# Do Work

Execute one complete **unit of work** end-to-end — understand it, plan it, build it, validate it, commit it — driven by **feedback loops**. Each phase is checked against a real signal (the user, the type checker, the tests, the build, your own review), and work never advances on a red signal. The execution engine that planning skills delegate to.

## Usage

```
/do-work [task, issue #, or plan reference]
```

Run it on a scoped task, or let another skill (e.g. `/prd-to-plan`, `/grind`) hand it a single slice to execute.

## How It Works

- Drives the work through a chain of loops (Understand → Plan → Implement → **Validate** → Self-review → Commit), each advancing only when its feedback signal is green
- **Cheapest signal first** in validation: typecheck/lint → tests → build → real behavior
- **Never advances on red** — a failing check feeds straight back into the next implement iteration
- Anti-faking guardrails: no deleting failing tests, no `--no-verify`, no scope creep
- Manual and delegatable — runs only when you invoke it or another skill explicitly hands it work

## When to Use

- Executing a single well-scoped task, issue, or plan slice from start to commit
- As the build-and-prove engine after a planning skill has decided *what* to do
- Any time you want "do the work, prove it works, commit it" with the loop discipline enforced

## Install

```bash
cp do-work.md ~/.claude/commands/
```
