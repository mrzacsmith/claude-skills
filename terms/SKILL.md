---
name: terms
description: Build and maintain a project's ubiquitous language — a CONTEXT.md glossary (plus ADRs and, for multi-repo projects, a context map). Interviews you to sharpen terminology, grounds terms in the code, and keeps the docs a living, drift-checked source of truth. Use to bootstrap or audit domain terminology, or when the user says "/terms".
---


Establish or refine the **ubiquitous language** for this repo: the shared vocabulary used by the code, the developers, and the domain experts. The output is a living `CONTEXT.md` glossary committed in the repo, so the whole team — and every future session — starts from shared understanding instead of re-explaining the domain.

This is the maintenance counterpart to `/grill-me`: grill-me *uses* the glossary while planning; `/terms` *builds and audits* it.

Interview one question at a time, always offering your recommended answer.

## What to do

### 1. Locate the context

- Look for an existing `CONTEXT.md` (repo root, then `docs/domain/`), `CONTEXT-MAP.md`, and `docs/adr/`.
- Decide whether this repo is **standalone** or part of a **multi-repo project** (independent repos that share a backend, schema, or core domain). Don't assume a topology — discover it: read an existing `CONTEXT-MAP.md` if there is one; otherwise look for sibling repos and ask me which are related and how. Never hard-code the number of repos or their names; the count is whatever the project currently has.
  - **Standalone:** a single `CONTEXT.md` at the repo root.
  - **Multi-repo:** map the actual shape rather than forcing one. Common shapes (and anything between):
    - **Hub-and-spoke** — one repo owns a shared kernel (schema, core entities, auth model); the others depend on it. The owner holds `SHARED-KERNEL.md` + `CONTEXT-MAP.md`; dependents reference shared terms by owner.
    - **Peers** — no single owner; each repo owns some terms and references others'. Any repo can host `CONTEXT-MAP.md`; each term is owned by whichever repo defines it.
  - Either way, every repo has its own `CONTEXT.md` for the terms it owns.

### 2. Mine the domain language

Explore the codebase for domain nouns and verbs: entity/model names, collection/table names, status enums, event names, recurring concepts in folder and file names. Build a candidate list. Distinguish **domain terms** (concepts unique to this project) from **general programming concepts** (timeouts, retries, generic utilities) — only the former belong in the glossary.

### 3. Sharpen through interview

For each fuzzy, overloaded, or duplicated term, grill me until it's precise:

- Propose one canonical term; list the rejected synonyms as aliases to avoid.
- Probe with concrete scenarios that force the boundary between two related concepts.
- Surface conflicts between what I say, what the glossary says, and what the code does.
- Be opinionated — pick the best word and say why.

### 4. Write it down (inline, as you go)

Update `CONTEXT.md` the moment each term settles — don't batch. Keep it a **glossary only**: define what each term *is*, not how it's implemented. Use the templates below. Group terms under subheadings when natural clusters emerge; otherwise a flat list is fine. End with a short example dialogue between a developer and a domain expert that shows the terms interacting.

### 5. Cross-repo wiring (multi-repo only)

- A term is **owned** by exactly one repo — the one where the concept originates — and defined in full there (that repo's `CONTEXT.md`, or a `SHARED-KERNEL.md` if the project has a hub that owns a shared kernel).
- Any other repo that uses the term gets a one-line entry naming the owner and this repo's relationship to it (reads / writes / extends) — e.g. *"**Cohort** — owned by `<owner-repo>`; this app only reads it."* Don't copy the full definition; point to the owner.
- Record or refresh `CONTEXT-MAP.md`: list **every** participating repo (however many), what each owns, and the relationships between them. Update it as repos are added or removed.

### 6. Architectural decisions

When a decision surfaces that is (1) hard to reverse, (2) surprising without context, and (3) the result of a real trade-off, offer to record an ADR in `docs/adr/`. All three must hold — otherwise skip it. Number sequentially (`0001-slug.md`).

### 7. Keep it living

- Stamp the bottom of `CONTEXT.md` with `Last reviewed: <date>` and a one-line changelog.
- Run a **drift check**: terms in the code but missing from the glossary, glossary terms no longer in the code (stale), and — for spokes — shared-term references that have diverged from the hub's `SHARED-KERNEL.md`. Report these and offer to fix.
- Treat glossary edits like code: commit them so the team picks them up. Don't push unless asked.

## Templates

### CONTEXT.md

```md
# {Repo / Context name} — Ubiquitous Language

{One or two sentences: what this context is and why it exists.}

## Language

**{Term}**:
{One or two sentences. Define what it IS, not what it does.}
_Avoid_: {rejected synonyms}

**{Term owned elsewhere — multi-repo}**:
{One line.} → Owned by **{owner repo}**; this repo {reads / writes / extends} it.

## Example dialogue

{A short dev <-> domain-expert exchange that uses the terms naturally and clarifies boundaries.}

---
Last reviewed: {YYYY-MM-DD} — {one-line changelog}
```

### CONTEXT-MAP.md

```md
# Context Map

## Repos
{One bullet per participating repo — however many there are.}
- [repo-a](path) — owns: {terms this repo is the source of truth for}
- [repo-b](path) — owns: {...}

## Relationships
{One bullet per relationship that matters; direction shows who depends on whom.}
- **repo-a -> repo-b**: {what flows, e.g. repo-b reads Cohort / Submission owned by repo-a}
- **repo-b <-> repo-c**: {shared types / events}
```

### docs/adr/NNNN-slug.md

```md
# {Short title of the decision}

{1–3 sentences: the context, what we decided, and why.}
```

## Rules

- `CONTEXT.md` is a glossary, not a spec or scratchpad — no implementation detail.
- Only include terms unique to this project's domain; leave out general programming concepts.
- Be opinionated and tight: one or two sentences per term, one canonical word per concept.
- In multi-repo projects, a term has exactly one owning repo — never redefine it elsewhere; reference the owner.
