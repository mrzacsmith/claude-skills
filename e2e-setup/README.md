# E2E Setup

Scaffold a production-grade Playwright E2E testing framework for any project. Auto-detects your stack, auth, routes, and selectors — asks only what it can't figure out.

## Usage

```
/e2e-setup
```

## Phases

### Phase 1: Auto-Detection

Detects before writing any code:

- **Package manager & framework** — npm/yarn/pnpm, Vite/Next/CRA/Remix/Astro/SvelteKit/Nuxt/Angular
- **Dev server port** — from config files or framework defaults
- **Auth provider** — Firebase, Supabase, Auth0, Clerk, NextAuth, Passport, JWT, or none
- **Database** — Firestore, Supabase, Prisma, MongoDB, PostgreSQL, SQLite, Drizzle
- **Emulators** — Firebase emulators, Supabase local, Docker services
- **Routes** — categorized as public, auth, protected, or admin
- **Auth UI selectors** — actual selectors from login/register components
- **User roles** — role hierarchy from guards, enums, and middleware

### Phase 2: User Questions

Confirms detection results and asks about:
- Deployment targets (local, staging, prod)
- Production URL
- Proposed test flows (auto-detected from routes/components)

### Phase 3: Scaffold

Creates a full test framework:

```
tests/
  e2e/
    flows/
      helpers/auth.{ext}          # loginUser, signupUser, logoutUser
      helpers/concurrency.{ext}   # assertAllFlowsSucceeded
      browse.flow.{ext}           # full user journeys (auto-detected)
      dashboard.flow.{ext}
    single/                       # one-at-a-time flow specs
    auth/                         # login, invalid creds, admin
    concurrent/                   # multi-user parallel tests
    smoke/                        # safe-for-prod health check
    helpers/env.{ext}             # target config
    helpers/setup.{ext}           # test user seeding
  fixtures/test-users.json
playwright.config.{ext}
```

Plus: package.json scripts, .gitignore entries, and GitHub Actions workflow.

### Phase 4: Validate

Runs `playwright test --list` and suggests a first sanity check.

## Key Architecture

- **Flow definitions** are plain functions, not test files — write the journey once, run it solo or N-at-a-time
- **Two layers**: helpers (building blocks) → flows (full journeys with assertions at every step)
- **Concurrent tests** use separate browser contexts with `Promise.allSettled` + failure assertion
- **Framework-aware** — correct dev server commands, env var prefixes, emulator wrappers

## Install

```bash
cp e2e-setup.md ~/.claude/commands/
```
