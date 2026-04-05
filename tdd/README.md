# TDD

Red-green-refactor test-driven development using vertical slices — one test at a time, not all tests first.

## Usage

```
/tdd
```

## Workflow

1. **Planning** — confirm interface changes, behaviors to test, and module design. Waits for approval.
2. **Tracer bullet** — one test, one implementation (RED → GREEN)
3. **Incremental loop** — repeat for each remaining behavior, one at a time
4. **Refactor** — only after all tests pass. Never refactor while RED.

## Philosophy

- Tests verify **behavior through public interfaces**, not implementation details
- Good tests survive refactors — they describe _what_ the system does, not _how_
- Vertical slices only: `test1→impl1, test2→impl2` — never `test1,test2,test3→impl1,impl2,impl3`
- Mock only at **system boundaries** (external APIs, time, randomness) — never mock your own code

## Includes

- Good vs bad test examples
- When to mock / when not to mock
- Per-cycle checklist
- Dependency injection patterns for testability

## Install

```bash
cp tdd.md ~/.claude/commands/
```
