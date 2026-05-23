---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.

## Domain awareness

Only applies when grilling about a codebase. If there's no codebase (a plan, a doc, a personal decision), skip this section and grill normally.

**At the start, look for the project's terminology docs:**

- `CONTEXT.md` — the project's glossary / ubiquitous language. Check the repo root, then `docs/domain/`.
- `CONTEXT-MAP.md` (and a `SHARED-KERNEL.md`, if the project has a hub) — present only in multi-repo projects. The map shows how this repo relates to the others; terms owned elsewhere are defined in their owner repo.

**If those docs exist, grill with them:**

- **Speak the documented language.** Use the canonical terms; don't invent synonyms for things that already have a name.
- **Challenge my words against the glossary.** If I use a term that conflicts with `CONTEXT.md`, or a vague/overloaded term, stop and resolve it: *"Your glossary defines X as Y, but you seem to mean Z — which is it?"*
- **Respect ownership.** If a term is owned by another repo (per `SHARED-KERNEL.md` / `CONTEXT-MAP.md`), its definition is fixed here — flag any attempt to redefine it locally.
- **Cross-check with code.** If I describe behavior the code contradicts, surface it.
- **Update the glossary inline.** The moment we sharpen or settle a term, write it into `CONTEXT.md` then and there — don't batch it. Keep `CONTEXT.md` a glossary only: what each term *is*, no implementation detail.

**If there's a codebase but no terminology docs:** once we've sharpened our first real term, offer — once, no nagging — to start the glossary with `/terms`.

**Where outputs go:** the plan and decisions from this session become **GitHub issues** (the work to do). Terminology stays in `CONTEXT.md`. A decision that's hard to reverse, surprising without context, and the result of a real trade-off becomes an ADR in `docs/adr/` — offer those sparingly.
