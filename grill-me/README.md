# Grill Me

Stress-test a plan or design through relentless interview. Walks down every branch of the decision tree, resolving dependencies one-by-one until reaching shared understanding.

## Usage

```
/grill-me
```

Provide your plan or design context, then get grilled.

## How It Works

- Asks questions **one at a time**
- Provides a **recommended answer** with each question
- Explores the codebase to answer questions it can resolve itself
- Keeps going until every branch of the decision tree is resolved

## When to Use

- Before starting a complex implementation
- When you want to find gaps in your design
- To stress-test architectural decisions
- When you say "grill me" about any plan

## Domain awareness

When grilling about a codebase, grill-me looks for the project's terminology docs and uses them:

- Reads `CONTEXT.md` (the glossary / ubiquitous language), plus `CONTEXT-MAP.md` and `SHARED-KERNEL.md` in multi-repo projects
- Challenges your language against the glossary and sharpens fuzzy/overloaded terms
- Updates `CONTEXT.md` inline as terms settle
- Routes outputs: plan/decisions → GitHub issues, terminology → `CONTEXT.md`, hard-to-reverse decisions → ADRs in `docs/adr/`

If no glossary exists, it offers to start one with [`/terms`](../terms/). With no codebase, it behaves exactly as before.

## Install

```bash
cp grill-me.md ~/.claude/commands/
```
