# Terms

Build and maintain a project's **ubiquitous language** — a living `CONTEXT.md` glossary, plus ADRs and (for multi-repo projects) a context map. The maintenance counterpart to `/grill-me`: grill-me *uses* the glossary, `/terms` *builds and audits* it.

## Usage

```
/terms
```

Run it inside a repo to bootstrap a glossary, or re-run it to audit and refine an existing one.

## How It Works

- Mines the codebase for domain terms, then **interviews you** to sharpen them
- Writes a glossary-only `CONTEXT.md`, committed in the repo for the whole team
- Supports multi-repo "wheels": a hub owns the shared kernel, spokes reference it by owner
- Records hard-to-reverse decisions as ADRs under `docs/adr/`
- Keeps docs living with a drift check (missing / stale / diverged terms)

## When to Use

- Starting a new project or feature area and you want shared language
- Onboarding a team to a domain
- After repeated `/grill-me` sessions where the same terms keep needing explanation
- To audit terminology that has drifted from the code

## Install

```bash
cp -R terms ~/.claude/skills/
```
