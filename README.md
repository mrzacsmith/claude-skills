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

## Prerequisites

Several skills interact with GitHub via the **GitHub CLI (`gh`)**. Skills that require it: `/write-prd`, `/triage-issue`, `/qa`, `/grind`, `/prd-to-issues`, `/prd-to-plan`.

**Install `gh`:**

```bash
# macOS
brew install gh

# Windows
winget install --id GitHub.cli

# Linux (Debian/Ubuntu)
sudo apt install gh

# Or see https://cli.github.com for all options
```

Then authenticate:

```bash
gh auth login
```

## GitHub Issue Defaults

Several skills automatically assign issues to `@me` (the authenticated `gh` user) when creating or listing issues. This affects: `/qa`, `/triage-issue`, `/prd-to-issues`, `/write-prd`, and `/grind`.

**Each team member** should add the following to their `~/.claude/CLAUDE.md` to set their GitHub username:

```markdown
# GitHub Issue Defaults

GitHub username: `<your-github-username>`

**All skills and workflows that interact with GitHub issues MUST follow these rules:**

- **Creating issues** (`/qa`, `/triage-issue`, `/prd-to-issues`, `/epic`, or any `gh issue create`): Always include `--assignee <your-github-username>` unless the user explicitly names a different assignee.
- **Selecting issues to work** (`/grind`, or any `gh issue list` used to pick up work): Always filter with `--assignee @me`. Do not work issues assigned to other people.
- **Closing/finishing issues** (`/grind`, `/commit`, PRs): Only close issues assigned to `<your-github-username>` unless explicitly told otherwise.
```

Replace `<your-github-username>` with your actual GitHub handle. This ensures each person's skills only create and pick up their own issues.

Other dependencies:

| Dependency | Required by | Install |
|------------|-------------|---------|
| `jq` | `/git-guardrails` | `brew install jq` |
| Node.js + npm | `/e2e-setup` | https://nodejs.org |

## Installation

Copy any skill's `.md` file into your `~/.claude/commands/` directory (global) or `.claude/commands/` in a project (project-scoped). Then use the corresponding `/command` in Claude Code.
