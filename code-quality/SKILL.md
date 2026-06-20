---
name: code-quality
description: >
  Bring a JavaScript repo up to the standard Code Quality Playbook baseline
  (Prettier, ESLint ratchet, husky + lint-staged, CI lint gate, optional JSDoc
  typecheck) idempotently — present pieces are skipped, missing ones are added —
  then optionally migrate the repo from npm to pnpm (lockfile import,
  packageManager pin, build-script allow-list, Firebase functions-framework fix,
  npm cleanup). Manual, global. Use only when the user runs /code-quality.
disable-model-invocation: true
allowed-tools: Bash Read Write Edit Grep AskUserQuestion
---

# /code-quality

Idempotently apply the **Code Quality Playbook** to the current repo, then
optionally migrate it from **npm → pnpm**. Manual and global: it runs only when
the user invokes it, and it must work on **any** JS repo, not just the one it was
written in.

The canonical knowledge lives in the two appendices at the bottom of this file —
read them when you need detail:

- **Appendix A — Code Quality Playbook** (Steps 1–5).
- **Appendix B — npm→pnpm migration** — the procedure and the three Firebase
  gotchas, proven end-to-end (cloud deploy verified).

**Core principle — the ratchet:** never big-bang. Add tooling, report baselines,
and let quality improve as files are touched. Do not mass-fix or mass-reformat.

---

## Operating rules

- **Idempotent.** Before adding anything, check if it already exists. Present →
  **skip** and say so. Missing → **add** it. Never overwrite a working config
  without showing the user first.
- **Analyze, don't assume ("code review").** Inspect the actual repo to decide
  what to add and to confirm each addition is correct for *this* layout — ESLint
  globs/globals must match where code lives; tsconfig `include` must be scoped to
  real source dirs; lint-staged globs must match real file types. A config that
  lints nothing is worse than none.
- **Multi-package aware.** A repo may have several `package.json` files (e.g.
  root frontend + `functions/` backend). Detect them all and apply the right
  config at the right level.
- **One question only, mid-run.** The pnpm migration is the single interactive
  decision. Everything else runs automatically and is reported.
- **Never run `firebase deploy` or any outward-facing/irreversible command
  yourself.** Stop at local checks; print deploy as a recommended manual step.
- **Never `npm audit fix --force`.**
- **Branch first if on the default branch** (`main`/`master`). Do not commit or
  push unless the user asks.

---

## Procedure

### Phase 0 — Detect

Run a quick survey (don't guess — look):

```bash
git rev-parse --is-inside-work-tree            # must be a git repo
git branch --show-current
node -v
find . -maxdepth 3 -name package.json -not -path '*/node_modules/*'
ls -a .github/workflows 2>/dev/null
cat firebase.json 2>/dev/null                  # Firebase functions project?
```

For each `package.json` found, note: is it a **browser/frontend** package (vite,
react, `"type":"module"`, `src/`), a **Node/CommonJS** package (`functions/`,
server, scripts), or **mixed**. Note the test runner (vitest/jest/none) and the
current package manager (which lockfile exists). Record findings; you'll report
them.

### Phase 1 — Playbook audit (idempotent)

Work through Steps 1–4 of **Appendix A**. For **each** piece, detect then act:

| Piece | Present check | If missing |
| --- | --- | --- |
| **Prettier** (Step 1) | `.prettierrc*`, `prettier` dep, `format` script | add `.prettierrc`, `.prettierignore`, deps, scripts |
| **ESLint ratchet** (Step 1) | `eslint.config.*`, `lint` script | add flat config with the **errors=real-bugs** floor; **set `files`/`globals` to this repo's actual dirs** |
| **husky + lint-staged** (Step 2) | `.husky/pre-commit`, `prepare:husky`, `lint-staged` config | install, `husky init`, write hook, add `lint-staged` globs matching real file types |
| **CI lint gate** (Step 3) | a workflow that lints changed files | add `.github/workflows/ci.yml` lint-changed-files job |
| **Typecheck** (Step 4, optional) | `tsconfig.json`/`jsconfig.json` + `typecheck` script | add `typescript` dev dep + scoped `tsconfig.json` + `typecheck` script **per package**; install `@types/node` where Node types are used |

**CI policy:** add CI jobs **non-blocking** (`continue-on-error: true` for
typecheck) and **only if missing**. If a CI lint gate already exists, leave it.
Do not flip anything to a hard merge-block.

**Multi-package typecheck:** mirror the pattern proven in the reference repo —
each package gets its own `tsconfig.json` + `typecheck` script, and the root gets
`typecheck:<pkg>` delegators plus a combined `typecheck`. Use `pnpm -C <dir>` /
`npm --prefix <dir>` per the active package manager.

After each addition, **run the tool once** (`lint`, `typecheck`) to capture the
baseline error count. Tune the ESLint hard-error floor so **errors = real bugs
only** (downgrade intentional-pattern rules to `warn`). Report counts; do **not**
fix the backlog.

### Phase 2 — Verify the additions ("code review")

Re-read what you added against the repo:

- ESLint actually matches files (`npx eslint .` lints real files, not zero).
- tsconfig `include` points at real source; `typecheck` runs.
- lint-staged globs match the languages present.
- husky hook is executable and uses the active package manager's runner.

Fix any mismatch now. Summarize: **skipped** (already present) vs **added** vs
**baseline counts**.

### Phase 3 — Offer pnpm migration (the one interactive decision)

Ask with `AskUserQuestion`: **"Migrate this repo from npm to pnpm?"**
(yes / no — note disk/strictness wins vs the per-property migration cost).

If **no** → skip to Phase 5.

If **yes** → follow **Appendix B** exactly. The shape:

1. **Pin** `"packageManager": "pnpm@<version>"` in every `package.json`
   (`corepack prepare pnpm@<v> --activate`; use the installed pnpm version).
2. **Import** each `package-lock.json` → `pnpm-lock.yaml` (`pnpm import` *before*
   deleting the npm lock — it reads it to preserve resolved versions).
3. **Clean + install:** remove the old npm `node_modules`, then `pnpm install`.
4. **Build-script allow-list (don't hardcode):** read the "Ignored build scripts"
   warning from the install output, add exactly those to
   `pnpm.onlyBuiltDependencies` in the relevant `package.json`, reinstall so they
   run. This field is read by the **cloud buildpack** too, so it must be
   committed.
5. **Firebase fix:** if this is a Firebase functions project, **declare
   `@google-cloud/functions-framework` as a direct dependency** in the functions
   package. npm hoists it transitively; pnpm does not, and the buildpack requires
   it explicitly. (This is the #1 cause of a pnpm Firebase deploy failure.)
6. **Convert npm references** in scripts/hooks to pnpm (`npm run` → `pnpm run`,
   `npm --prefix X` → `pnpm -C X`, `npx` → `pnpm exec`, husky hook, CI `npm ci`
   → `pnpm install --frozen-lockfile` with `pnpm/action-setup`).

### Phase 4 — Cleanup (pnpm path only)

Remove what npm leaves behind, now redundant under pnpm:

- delete every `package-lock.json`
- ensure `pnpm-lock.yaml` files exist and are staged
- ensure `firebase-debug.log` / `*-debug.log` are gitignored
- leave a **working** pnpm `node_modules` (do not delete it — the repo must still
  be runnable).

### Phase 5 — Final verification + report

Run the local checks for the active package manager and report results:

- `lint` (every package)
- `test` (every package with a test script)
- `build` (if present — confirms native deps like `sharp` built)
- `typecheck` (if set up)

**Firebase projects:** print, do **not** run:
`firebase deploy --only functions --project <id>` — the true end-to-end check for
a pnpm migration.

Final report:
- **Skipped** (already present) / **Added** / **Migrated to pnpm? y/n**
- Baseline lint + typecheck counts (note: on-demand, not a hard gate yet)
- Files changed; branch name
- Next steps (commit/PR; for Firebase, the deploy command to verify)

---

## Done when

The repo has the playbook baseline (idempotently), optionally pnpm with all three
gotchas handled, npm leftovers cleaned, local checks run and reported, and nothing
committed/deployed without the user asking.

---

# Appendix A — Code Quality Playbook

A step-by-step guide to add the **same** linting / formatting / type-checking
process to any JavaScript repo.

It's built around one idea — the **ratchet**:

> Don't try to clean up the whole codebase at once. Enforce quality only on the
> files each commit/PR touches, so the code monotonically improves without a
> big-bang cleanup. CI is the team-wide backstop; the local hook is the fast
> feedback loop.

## What you get

| Layer           | When it runs                    | What it does                                                           |
| --------------- | ------------------------------- | ---------------------------------------------------------------------- |
| Pre-commit hook | on `git commit` (after install) | Auto-fixes + lints **staged** files; blocks commit on unfixable errors |
| CI lint gate    | on every PR (GitHub Actions)    | Lints **changed** files; new errors = red                             |
| CI typecheck    | on every PR                     | `tsc --noEmit` over JSDoc-typed JS (optional, non-blocking first)      |

**Core** = Steps 1–3 (works in any JS repo). **Optional** = Steps 4–5 (type
safety + shared-code pattern; adopt when it pays off).

> Throughout, **adapt the globs and globals to the repo**: a browser app uses
> `globals.browser`, a Node service uses `globals.node`, a mixed repo scopes both
> by directory.

---

## Step 1 — ESLint + Prettier (the ratchet config)

**1a. Install:** `eslint @eslint/js prettier eslint-config-prettier globals` (dev).

**1b. Create `eslint.config.js`** (flat config). Adjust `files`/`globals` to the
repo's layout:

```js
import js from '@eslint/js'
import globals from 'globals'
import prettier from 'eslint-config-prettier'

export default [
  { ignores: ['node_modules/**', 'dist/**', 'build/**', 'coverage/**'] },

  js.configs.recommended,

  // High-signal bug catchers as hard ERRORS; noisy/stylistic rules as WARN so
  // legacy code doesn't block the ratchet. This split is the whole trick.
  {
    rules: {
      eqeqeq: ['error', 'smart'],
      'no-const-assign': 'error',
      'no-dupe-keys': 'error',
      'no-dupe-args': 'error',
      'no-unreachable': 'error',
      'no-func-assign': 'error',
      'no-cond-assign': ['error', 'always'],

      'no-unused-vars': ['warn', { args: 'none', ignoreRestSiblings: true }],
      'no-empty': 'warn',
      'no-constant-condition': ['warn', { checkLoops: false }],
      'no-control-regex': 'warn',
      'no-useless-escape': 'warn',
      'no-useless-assignment': 'warn',
      'no-prototype-builtins': 'warn',
    },
  },

  // Browser source — set the path glob to wherever frontend code lives.
  {
    files: ['src/**/*.js'],
    languageOptions: {
      ecmaVersion: 'latest',
      sourceType: 'module',
      globals: { ...globals.browser, ...globals.es2024 },
    },
  },

  // Node code (server, scripts, config) — adjust globs to the repo.
  {
    files: ['server/**/*.js', 'scripts/**/*.{js,mjs,cjs}', '*.{js,mjs,cjs}'],
    languageOptions: {
      ecmaVersion: 'latest',
      sourceType: 'module',
      globals: { ...globals.node, ...globals.es2024 },
    },
  },

  { files: ['**/*.cjs'], languageOptions: { sourceType: 'commonjs' } },

  prettier, // disable formatting rules that fight Prettier — must be LAST.
]
```

> **Tune the hard-error floor.** Run `npm run lint` once and look at what's an
> error. Anything that's an _intentional pattern_ in this codebase should be a
> `warn`. Goal: **errors = real bugs only**, so the CI gate is trustworthy.

**1c. `.prettierrc`:**

```json
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "always"
}
```

**1d. `.prettierignore`:**

```
node_modules
dist
build
coverage
package-lock.json
pnpm-lock.yaml
*.min.js
*.min.css
```

**1e. Scripts:**

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
}
```

**1f. Run and tune.** Expect noise on a legacy repo. **Do not** mass-fix. Make
sure the only remaining **errors** are genuine bugs. Note the error count.

---

## Step 2 — husky + lint-staged (auto-enforce on commit)

**2a. Install + init:** `husky lint-staged` (dev), then `npx husky init` (creates
`.husky/pre-commit` and the `"prepare": "husky"` script that activates the hook
on install).

**2b. `.husky/pre-commit`:** `npx lint-staged` (or `pnpm exec lint-staged` under
pnpm).

**2c. `lint-staged` config** (adjust file types to the repo):

```json
{
  "lint-staged": {
    "*.{js,mjs,cjs}": ["eslint --fix", "prettier --write"],
    "*.{css,html,json,md}": ["prettier --write"]
  }
}
```

**2d. Verify** fixable→auto-fixed, unfixable→blocks commit, then clean up the
test files.

**2e. Document** in README: run install to activate; `--no-verify` to bypass in
emergencies.

---

## Step 3 — CI lint gate (the team-wide backstop)

Lint **only changed files** so the legacy backlog never blocks a PR.

**3a. `.github/workflows/ci.yml`:**

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lint:
    name: Lint (changed files)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # full history so we can diff against the PR base

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm # (use pnpm cache + pnpm/action-setup under pnpm)

      - name: Install dependencies
        run: npm ci # (pnpm install --frozen-lockfile under pnpm)

      - name: Lint changed JS files
        env:
          BASE_SHA: ${{ github.event.pull_request.base.sha }}
        run: |
          set -euo pipefail
          if [ -z "$BASE_SHA" ]; then
            BASE_SHA="$(git rev-parse HEAD^ 2>/dev/null || echo '')"
          fi
          if [ -z "$BASE_SHA" ]; then
            echo "No base ref; linting whole tree."; npx eslint .; exit $?
          fi
          FILES="$(git diff --name-only --diff-filter=ACMR "$BASE_SHA" HEAD -- '*.js' '*.mjs' '*.cjs' \
            | grep -vE '^(node_modules|dist|build)/' || true)"
          if [ -z "$FILES" ]; then echo "No JS changes to lint."; exit 0; fi
          echo "Linting:"; echo "$FILES"
          echo "$FILES" | xargs npx eslint
```

**3b. (Optional, paid plan) Make it required** via a branch ruleset that requires
the `Lint (changed files)` status check. Without it the check still runs and
shows red/green; it just won't physically block merges.

---

## Step 4 — Type checking with JSDoc + `checkJs` (optional)

Build-time type checking with **no file renames** and **no runtime change** (types
are erased). Best targeted at backend/business logic.

**4a. Install:** `typescript` (dev), plus `@types/node` in packages that use Node
globals (`process`, `Buffer`, `require`, `__dirname`).

**4b. `tsconfig.json`** per package — **scope `include` tightly**, widen later.
Node/CommonJS package:

```json
{
  "compilerOptions": {
    "allowJs": true,
    "checkJs": true,
    "noEmit": true,
    "strict": false,
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "skipLibCheck": true,
    "lib": ["ES2022"],
    "types": ["node"]
  },
  "include": ["lib/**/*.js", "index.js"],
  "exclude": ["**/node_modules/**", "**/__tests__/**", "**/*.test.js"]
}
```

Browser/React package — drop `"types": ["node"]` (it would exclude the React
types it needs), and use:

```json
{
  "compilerOptions": {
    "allowJs": true,
    "checkJs": true,
    "noEmit": true,
    "strict": false,
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "jsx": "react-jsx",
    "skipLibCheck": true,
    "lib": ["ES2022", "DOM", "DOM.Iterable"]
  },
  "include": ["src/**/*.js", "src/**/*.jsx"],
  "exclude": ["**/node_modules/**", "**/*.test.js"]
}
```

**4c. Scripts.** Single package: `{ "typecheck": "tsc --noEmit" }`. Multi-package
(root delegates):

```json
{
  "typecheck": "npm run typecheck:frontend && npm run typecheck:functions",
  "typecheck:frontend": "tsc --noEmit",
  "typecheck:functions": "npm --prefix functions run typecheck"
}
```

(Under pnpm: `pnpm run …` and `pnpm -C functions run typecheck`.)

**4d. Run, then drive the baseline toward zero.** Fix findings at the boundary
(coerce inputs, annotate option objects), not by loosening types. checkJs on a
never-checked codebase produces a large baseline (often hundreds) — mostly
implicit-`unknown` on un-annotated data and async-return JSDoc nits, not real
bugs. Keep it **on-demand** until the baseline is driven down; only then make CI
blocking.

**4e. CI job — start non-blocking, flip to blocking once green:**

```yaml
typecheck:
  name: Typecheck
  runs-on: ubuntu-latest
  continue-on-error: true # remove once `npm run typecheck` is clean
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with: { node-version: 22, cache: npm }
    - run: npm ci
    - run: npm run typecheck
```

---

## Step 5 — Sharing code across a build boundary (optional, advanced)

Use when the **same logic must live in two places** that can't import each other
— e.g. an ESM browser bundle and a self-contained CommonJS service that deploys
alone. Pattern: **one source of truth → generated copy → drift test.**

1. **Source of truth** in the importing side (e.g. `src/shared/core.js`, ESM),
   plain top-level `export const`/`export function` (no imports).
2. **Generator** (`scripts/build-core.mjs`) does an ESM→CJS text transform and
   writes `<other-dir>/core.generated.js`. Wire into `prebuild` / `predeploy`.
3. **Exclude the generated file** from eslint + prettier.
4. **Drift test** regenerates in-memory and asserts equality with the committed
   copy.

> Lower-effort alternative: keep both copies but add a test asserting the shared
> constants are identical.

---

## Gotchas

- **Activation needs an install, not `git pull`.** The hook installs via the
  `prepare` script on install. Put a one-line note in the README.
- **`npm ci` may skip `prepare`.** Devs use `npm install` locally; `npm ci` is for
  CI (where the hook is irrelevant — the Actions gate covers CI).
- **Hooks are local and bypassable** (`--no-verify`). **CI is the real backstop.**
- **Required checks (hard merge-block) need a paid plan** for private repos.
- **Don't big-bang.** Let the ratchet clean files as they're touched; fix genuine
  bugs in small, separate PRs.

## Quick checklist per repo

- [ ] Step 1: `eslint.config.js` + `.prettierrc` + `.prettierignore` + scripts; hard-error floor = real bugs only
- [ ] Step 2: husky + lint-staged; verified fix/block behavior; README note
- [ ] Step 3: `ci.yml` lint gate; (optional) required-check ruleset
- [ ] Step 4 (optional): `tsconfig.json` + `typecheck` script + CI job (non-blocking)
- [ ] Step 5 (optional): shared-source + generator + drift test

---

# Appendix B — npm → pnpm migration (proven end-to-end, incl. Firebase Functions cloud deploy)

Why pnpm: strict non-flat `node_modules` (catches phantom/hoisted dependencies),
a content-addressable global store (huge disk + speed win across many repos),
deterministic installs, first-class workspaces. The strictness is the reliability
win — it surfaces deps a build was relying on without declaring.

This procedure was validated on a two-package Firebase repo (Vite React frontend +
Cloud Functions Gen 2): all local checks green, **and** `firebase deploy --only
functions` deployed ~60 functions from `pnpm-lock.yaml` via the cloud buildpack.

---

## Preconditions

- `corepack --version` works (ships with Node 16.9+). Get the pnpm version with
  `pnpm --version` (corepack provides it) — pin **that exact version**.
- Do this on a branch, not the default branch.

## Procedure (per repo; repeat per package where noted)

### 1. Pin the package manager (corepack)

```bash
corepack prepare pnpm@<version> --activate
```

Add to **every** `package.json`: `"packageManager": "pnpm@<version>"`. This makes
corepack (local) and the **cloud buildpack** agree on the pnpm version, and is how
the buildpack detects that the project uses pnpm.

### 2. Import the lockfile (preserve resolved versions)

Per package, **before** deleting the npm lock:

```bash
pnpm import          # reads package-lock.json -> writes pnpm-lock.yaml
```

### 3. Clean install

Per package:

```bash
rm -f package-lock.json
rm -rf node_modules
pnpm install
```

### 4. Build-script allow-list (do NOT hardcode — read the warning)

pnpm 10 blocks dependency postinstall/build scripts by default (a supply-chain
guard). The install prints:

```
╭ Warning ─ Ignored build scripts: sharp@..., protobufjs@..., @firebase/util@... ╮
```

Take **exactly those package names** and add them to the package that owns them:

```json
{ "pnpm": { "onlyBuiltDependencies": ["@firebase/util", "esbuild", "protobufjs", "sharp", "unrs-resolver"] } }
```

Then `pnpm install` again so the scripts run. Verify natives load
(`node -e "require('sharp')"`, etc.).

> This field is committed and **read by the cloud buildpack** too — without it the
> cloud install also skips the scripts. Set it in the package that actually
> depends on each (frontend usually needs `sharp`/`esbuild`; a Firebase functions
> package needs `@firebase/util`/`protobufjs`/`unrs-resolver`).

### 5. Firebase Functions fix — THE #1 gotcha

If this is a Firebase functions project (`firebase.json` has a `functions` block /
the package depends on `firebase-functions`), the cloud deploy fails with:

> *"This project is using pnpm but you have not included the Functions Framework
> in your dependencies. Please add it by running: `pnpm add
> @google-cloud/functions-framework`."*

Cause: `@google-cloud/functions-framework` is a **transitive** dep of
`firebase-functions`. npm's flat `node_modules` hoists it to the top so the
buildpack finds it; pnpm does not hoist it, so it must be **declared explicitly**.
Fix in the functions package:

```bash
pnpm add @google-cloud/functions-framework
```

(This is a genuine win, not a regression — pnpm exposed a phantom dependency the
deploy was silently relying on.)

### 6. Convert npm references to pnpm

- `package.json` scripts: `npm run X` → `pnpm run X`; `npm --prefix DIR run X` →
  `pnpm -C DIR run X` (pnpm has no `--prefix`); `npx Y` → `pnpm exec Y`.
- `.husky/*` hooks: `npx lint-staged` → `pnpm exec lint-staged`.
- CI workflows: add `pnpm/action-setup`, switch `cache: npm` → `cache: pnpm`, and
  `npm ci` → `pnpm install --frozen-lockfile`.

### 7. Cleanup

- delete every `package-lock.json`
- keep/stage every `pnpm-lock.yaml`
- gitignore `*-debug.log` (e.g. `firebase-debug.log` from a failed deploy)
- leave a **working** pnpm `node_modules` — the repo must stay runnable.

### 8. Verify

- local: `lint`, `test`, `build`, `typecheck` per package.
- Firebase: **print, don't run** `firebase deploy --only functions --project <id>`
  as the true end-to-end check. (A transient `429 Quota Exceeded — Per project
  mutation requests per minute` while deploying many functions is rate-limiting,
  not a pnpm problem; the CLI auto-retries.)

---

## Quick checklist

- [ ] `packageManager` pinned in every `package.json`
- [ ] `pnpm import` → `pnpm-lock.yaml` per package; npm lock deleted
- [ ] `pnpm.onlyBuiltDependencies` set from the actual "Ignored build scripts" list
- [ ] Firebase: `@google-cloud/functions-framework` is a direct dependency
- [ ] npm refs converted (scripts, husky, CI) to pnpm
- [ ] local checks green; deploy command printed for manual verification
