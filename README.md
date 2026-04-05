# claude-skills

A collection of 13 Claude Code custom skills (slash commands) for structured development workflows.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [commit](./commit/) | `/commit` | Emoji-prefixed git commits with optional push |
| [catchup](./catchup/) | `/catchup` | Summarize uncommitted changes in the working tree |
| [epic](./epic/) | `/epic` | Generate structured epics with interactive approval phases |
| [write-prd](./write-prd/) | `/write-prd` | Write a PRD through interview, file as GitHub issue |
| [tdd](./tdd/) | `/tdd` | Red-green-refactor TDD with vertical slices |
| [prd-to-plan](./prd-to-plan/) | `/prd-to-plan` | Convert a PRD into phased vertical-slice implementation plan |
| [triage-issue](./triage-issue/) | `/triage-issue` | Investigate a bug, find root cause, file GitHub issue with TDD fix plan |
| [qa](./qa/) | `/qa` | Rapidly file well-structured GitHub issues |
| [grind](./grind/) | `/grind` | Work through open issues with TDD, feature branches, and PRs |
| [prd-to-issues](./prd-to-issues/) | `/prd-to-issues` | Break a PRD into vertical-slice GitHub issues |
| [git-guardrails](./git-guardrails/) | `/git-guardrails` | Set up hooks to block dangerous git commands |
| [grill-me](./grill-me/) | `/grill-me` | Stress-test a plan or design through relentless interview |
| [e2e-setup](./e2e-setup/) | `/e2e-setup` | Scaffold Playwright E2E testing with auto-detection |

## Installation

Copy any skill's `.md` file into your `~/.claude/commands/` directory (global) or `.claude/commands/` in a project (project-scoped). Then use the corresponding `/command` in Claude Code.
