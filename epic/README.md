# Epic

Generate structured development epics with a 7-phase interactive approval workflow. Breaks features into tasks and subtasks, then executes them one at a time with commits.

## Usage

```
/epic add user authentication with JWT
```

## Phases

1. **Discovery** — scans project, asks 2-4 clarifying questions
2. **Epic Overview** — presents summary, scope, affected areas (approve/revise/exit)
3. **Task Breakdown** — detailed tasks with conventional commit messages (approve/revise/exit)
4. **Subtask Expansion** — 3-5 subtasks per task (approve/revise/exit)
5. **Save Epic File** — writes `EPIC_[NAME].md` to track progress
6. **Execution** — implements tasks one at a time, commits after each, updates the epic file
7. **Completion** — runs review checkpoint, marks epic complete

## Key Features

- Every phase gate requires your approval before proceeding
- Progress saved to `EPIC_[NAME].md` after each task
- Resume support — running `/epic` with an existing epic file picks up where you left off
- Uses conventional commits (`feat`, `fix`, `chore`, etc.) with configurable scope
- Each task is atomic and independently committable

## Install

```bash
cp epic.md ~/.claude/commands/
```
