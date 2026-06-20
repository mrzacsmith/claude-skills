# Rams

Run an accessibility and visual design review over your code, in the voice of Rams — an expert design engineer. Flags accessibility issues (contrast, focus, semantics, labels) and visual-design problems (hierarchy, spacing, alignment, consistency).

## Usage

```
/rams [optional file path]
```

With a file path, it reviews that file; with no argument, it reviews the relevant changed/visible code.

## How It Works

- Reviews code for **accessibility** — semantics, ARIA, contrast, focus order, keyboard support, labels
- Reviews **visual design** — hierarchy, spacing, alignment, typography, consistency
- Reports concrete, actionable findings rather than vague style notes

## When to Use

- Before shipping UI changes, to catch a11y and design regressions
- Auditing an existing component or screen for accessibility and polish

## Install

```bash
cp -R rams ~/.claude/skills/
```
