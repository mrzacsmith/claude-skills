---
description: Set up Playwright E2E testing framework for any project — auto-detects stack, auth, routes, and selectors
allowed-tools: Read, Glob, Grep, Bash, Write, Edit, Agent, AskUserQuestion
---

# Playwright E2E Testing Setup (Universal)

You are setting up a production-grade Playwright E2E testing framework for the current project. You must auto-detect as much as possible and ask the user only what you can't figure out.

---

## Phase 1: Auto-Detection (Do All of This BEFORE Writing Any Code)

### 1.1 Detect Package Manager & Framework

Read `package.json` (and `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` if they exist) to determine:

- **Package manager**: npm, yarn, or pnpm
- **Framework**: Vite, Next.js, CRA, Remix, Astro, SvelteKit, Nuxt, Angular, or plain Node/Express
- **Language**: JavaScript or TypeScript (check for `tsconfig.json`, `.ts`/`.tsx` files)
- **Module system**: ESM (`"type": "module"`) or CJS

### 1.2 Detect Dev Server Port

Check these sources in order to find the dev server port:
- `vite.config.*` → `server.port`
- `next.config.*` → check for custom port
- `angular.json` → `serve.options.port`
- `package.json` scripts → look for `--port` flags in `dev`/`start` scripts
- Framework defaults: Vite=5173, Next=3000, CRA=3000, Angular=4200, Remix=3000, Astro=4321, SvelteKit=5173, Nuxt=3000

Record this as `DEV_PORT`. The **test port** will be `DEV_PORT + 26` (e.g., 5173→5199, 3000→3026) to avoid conflicts with a running dev server. This is configurable via `TEST_PORT` env var.

### 1.3 Detect Dev Server Start Command

Read the `dev` or `start` script from `package.json`. This is what Playwright's `webServer` block will run, with the port overridden to the test port.

### 1.4 Detect Auth Provider

Search for imports and usage of:
- **Firebase Auth**: `firebase/auth`, `connectAuthEmulator`
- **Supabase Auth**: `@supabase/supabase-js`, `supabase.auth`
- **Auth0**: `@auth0/auth0-react`, `@auth0/nextjs-auth0`
- **Clerk**: `@clerk/clerk-react`, `@clerk/nextjs`
- **NextAuth/Auth.js**: `next-auth`, `@auth/core`
- **Passport**: `passport`, `passport-local`
- **Custom/JWT**: manual JWT handling, `jsonwebtoken`
- **None**: no auth detected

Record which provider and which auth services are initialized (email/password, Google SSO, magic link, etc.).

### 1.5 Detect Database / Backend

Search for:
- **Firestore**: `firebase/firestore`, `connectFirestoreEmulator`
- **Supabase DB**: `supabase.from()`
- **Prisma**: `@prisma/client`
- **MongoDB/Mongoose**: `mongoose`, `mongodb`
- **PostgreSQL**: `pg`, `postgres`
- **SQLite**: `better-sqlite3`, `sql.js`
- **Drizzle**: `drizzle-orm`
- **API-only**: REST/GraphQL calls to external services

### 1.6 Detect Emulator / Local Dev Setup

Check for:
- `firebase.json` → emulator config (which emulators, which ports)
- `.env.local`, `.env.development` → local service URLs
- `docker-compose.yml` → local database/service containers
- `supabase/config.toml` → Supabase local config
- Any seed scripts that already exist

Record which emulators/local services are available and their ports.

### 1.7 Detect Router & Routes

Find the router configuration:
- **React Router**: search for `createBrowserRouter`, `<Route`, `<Routes`, a routes config file
- **Next.js**: read `app/` or `pages/` directory structure
- **SvelteKit**: read `src/routes/` directory structure
- **Nuxt**: read `pages/` directory structure
- **Angular**: search for `RouterModule`, route config files
- **Vue Router**: search for `createRouter`, route config

Categorize discovered routes into:
- **Public** (no auth required)
- **Auth pages** (login, register, forgot password)
- **Protected** (requires auth)
- **Admin** (requires elevated role)

### 1.8 Detect Auth UI Selectors

Find the login and registration components by searching for:
- Files named `login`, `signin`, `sign-in`, `auth` in component/page directories
- Files named `register`, `signup`, `sign-up` in component/page directories
- Form elements with email/password inputs

Read those files and extract the **actual selectors**:
- Input fields: look for `placeholder`, `name`, `id`, `data-testid`, `aria-label` attributes
- Submit buttons: look for button text, `type="submit"`, `data-testid`
- Error messages: look for error display patterns
- Navigation elements: look for logout buttons, user menus

### 1.9 Detect User Roles

Search for role checks in the codebase:
- Route guards / protected route components
- Role constants or enums
- Middleware that checks roles
- Admin route wrappers

Record the role hierarchy (e.g., user → admin → owner).

---

## Phase 2: Ask the User

After detection, present a summary of what you found and ask:

1. **"Do you have a staging or dev deployment, or just local + prod?"**
   - Only configure targets the user confirms
   - If no staging: do NOT create staging config, scripts, or env vars

2. **"What is your production URL?"** (if not detectable from config files, hosting config, or `package.json` homepage field)

3. **Present the flows you auto-detected from route groups and components:**
   - Public routes → browse flow
   - Registration route → registration flow
   - Primary authenticated feature → dashboard/app flow
   - CRUD operations → CRUD flow (only if the project has create/edit/delete)
   - Settings/profile → settings flow (only if present)
   - Show each proposed flow and what it covers, then ask:
     - "Here are the flows I detected — confirm or modify."
     - "Which flows matter most for concurrent testing?"
     - "Are there any critical user paths I missed?" (auto-detection can't catch everything — payment flows, multi-step wizards, third-party OAuth redirects may need manual input)

4. **Confirm your detection results** — show what you found for framework, auth, database, routes, selectors, and proposed flows. Let the user correct anything wrong.

---

## Phase 3: Scaffold

### 3.1 Install Dependencies

```bash
{pkg_manager} install -D @playwright/test
npx playwright install chromium
```

### 3.2 Directory Structure

Create this structure (adapt filenames to the project's language — `.ts` if TypeScript, `.js` if JavaScript):

```
tests/
  e2e/
    flows/
      helpers/
        auth.{ext}           # Building blocks: loginUser, signupUser, logoutUser, waitForAuth
        concurrency.{ext}    # assertAllFlowsSucceeded utility
      browse.flow.{ext}      # Full journey: public page navigation with assertions at every step
      registration.flow.{ext} # Full journey: signup through onboarding
      dashboard.flow.{ext}   # Full journey: login → primary app usage → logout
      # (additional flows auto-detected from Phase 1 route/component scan)
    single/
      browse.spec.{ext}           # Imports browse flow, runs once
      registration.spec.{ext}     # Imports registration flow, runs once
      dashboard.spec.{ext}        # Imports dashboard flow, runs once
    auth/
      user-login.spec.{ext}
      invalid-creds.spec.{ext}
      [admin-login.spec.{ext}]    # Only if admin role detected
    concurrent/
      multi-user.spec.{ext}       # Imports flows, runs N in parallel with mixed traffic
    smoke/
      health.spec.{ext}
    helpers/
      env.{ext}            # Environment/target config
      setup.{ext}          # Test user seeding (if emulators/local DB available)
  fixtures/
    test-users.json
playwright.config.{ext}
```

**Key structural principle:** Flow definitions (`flows/*.flow.{ext}`) are plain exported functions, not test files. They take a `page` (and optional params) and walk through a complete user journey with assertions at every step. Test files in `single/` and `concurrent/` import these flows and run them in different execution contexts. Write the walkthrough once, run it alone to verify correctness, then run N of them to verify concurrency.

### 3.3 Environment Config (`tests/e2e/helpers/env.{ext}`)

Create an env config that:
- Reads `TEST_TARGET` env var — defaults to `local`
- Reads `TEST_PORT` env var, falling back to the computed test port
- Reads `CONCURRENT_USERS` env var for parallel browser contexts in concurrent tests, defaulting to `5` (safe for local dev machines; scale up in CI)
- For each confirmed target (local, staging, prod): returns `{ baseURL, useEmulators, project, concurrentUsers }`
- Exports a `getConfig()` function

**Only include targets the user confirmed.**

### 3.4 Playwright Config (`playwright.config.{ext}`)

- `testDir`: `tests/e2e`
- `timeout`: 60000 (concurrent flows need headroom)
- `expect.timeout`: 10000
- Screenshots: only on failure
- Video: retain on failure
- Trace: retain on first retry
- **Reporter**: `'html'` by default. When `E2E_LOG` env var is set, use multi-reporter: `[['list'], ['html'], ['json', { outputFile: 'e2e-logs/<ISO-timestamp>.json' }]]`. Timestamp uses `new Date().toISOString().replace(/[:.]/g, '-')`.
- **`webServer`**: start the project's dev server on the test port. Set any emulator env vars when target is local. Use `reuseExistingServer: false` always.
- **Projects** (only include confirmed targets):
  - `local` — baseURL from env config, runs all tests
  - `staging` — only if user confirmed staging exists
  - `prod` — baseURL from prod URL, runs all tests
  - `smoke-only` — prod baseURL, testDir limited to `tests/e2e/smoke/`

**Framework-specific webServer adjustments:**
- **Vite**: `npx vite --port {TEST_PORT}`
- **Next.js**: `npx next dev -p {TEST_PORT}`
- **CRA**: `PORT={TEST_PORT} npx react-scripts start`
- **Angular**: `npx ng serve --port {TEST_PORT}`
- **Remix**: `PORT={TEST_PORT} npx remix dev`
- **Astro**: `npx astro dev --port {TEST_PORT}`
- **SvelteKit**: `npx vite dev --port {TEST_PORT}`
- **Nuxt**: `npx nuxt dev --port {TEST_PORT}`

**Framework-specific emulator env vars:**
- **Vite projects**: prefix with `VITE_` (e.g., `VITE_USE_EMULATORS=true`)
- **Next.js**: prefix with `NEXT_PUBLIC_` (e.g., `NEXT_PUBLIC_USE_EMULATORS=true`)
- **CRA**: prefix with `REACT_APP_` (e.g., `REACT_APP_USE_EMULATORS=true`)
- **Angular**: use `environment.ts` or env vars
- **Others**: use plain env vars

### 3.5 Emulator / Local Service Connection

Based on what was detected:

**Firebase**: Add conditional emulator connection in the Firebase init file if not already present. Only connect emulators that are actually configured. Use ports from `firebase.json`. Prefer port 8085 for Firestore if configuring fresh (8080 conflicts with other services).

**Supabase**: Point to local Supabase instance (default `http://localhost:54321`).

**Docker services**: Document that `docker-compose up` is needed before tests.

**No local services**: Skip this step, tests will run against live services.

**Important**: Minimize changes to source code. Only add the emulator/local connection conditional if it doesn't already exist.

### 3.6 Flow Definitions (two layers)

Flows are the core abstraction of this framework. There are two layers:

#### Layer 1: Flow Helpers (`tests/e2e/flows/helpers/auth.{ext}`)

Low-level building blocks that other flows compose. These perform a single action and return — they are **not** full journeys. Using the **actual selectors detected in Phase 1**, create:

- **`loginUser(page, email, password)`**: Navigate to login page, fill credentials using real selectors, submit, wait for navigation away from login
- **`signupUser(page, { name, email, password, ...fields })`**: Navigate to register page, fill all fields using real selectors, submit (note any post-signup redirects like payment)
- **`logoutUser(page)`**: Find and click logout using real selectors, wait for redirect to login
- **`waitForAuth(page)`**: Wait for auth state to resolve — either dashboard content appears or auth page appears

**Use the exact selectors from the actual components.** Do not guess or use generic selectors. If a selector can't be determined, leave a `// TODO: update selector` comment.

#### Layer 2: Flow Definitions (`tests/e2e/flows/*.flow.{ext}`)

Full user journeys that take a `page` (and optional params) and walk through a complete path **with assertions at every step**. Not just "click and move on" — each step waits, then verifies the UI is correct before proceeding.

The specific flows to generate are **auto-detected from Phase 1 scanning** of routes and components. Do not prescribe fixed flows — detect what the project has and generate flows that match. Common patterns:

**Browse flow** (`browse.flow.{ext}`) — for projects with public pages:
- Land on home page → assert key elements render (hero, nav, CTA) → navigate to another public page → assert content renders → navigate to a third → assert no console errors throughout
- Every navigation step has an assertion before proceeding

**Registration flow** (`registration.flow.{ext}`) — for projects with user signup:
- Land on home → click registration link → assert register form renders → fill form → submit → assert redirect to expected post-signup destination → assert welcome/onboarding content visible
- Uses `signupUser` helper internally

**Dashboard/primary app flow** (`dashboard.flow.{ext}`) — for projects with authenticated areas:
- Uses `loginUser` helper to authenticate → assert main app view renders → navigate to primary feature → assert feature content loads → navigate to secondary feature → assert it loads → logout → assert return to public/login page
- Uses `loginUser` and `logoutUser` helpers internally

**Generate flows based on what the project actually has.** A CRUD app gets a CRUD flow. A read-only dashboard gets a dashboard flow. A content site gets a browse flow. A project with settings gets a settings flow. The key contract for every flow:

```javascript
// Every flow follows this contract:
// - Takes a page and optional params
// - Walks through a complete linear journey
// - Asserts at EVERY step before proceeding
// - Throws on any assertion failure
export async function browseFlow(page) {
  await page.goto('/');
  await expect(page.locator('...')).toBeVisible(); // assert before moving on
  
  await page.click('...');
  await expect(page.locator('...')).toBeVisible(); // assert before moving on
  
  // ... every step follows this pattern
}
```

#### Concurrency Utility (`tests/e2e/flows/helpers/concurrency.{ext}`)

Export a utility for asserting on `Promise.allSettled` results — this prevents concurrent tests from silently swallowing failures:

```javascript
import { expect } from '@playwright/test';

export function assertAllFlowsSucceeded(results) {
  const failures = results
    .map((r, i) => ({ ...r, index: i }))
    .filter(r => r.status === 'rejected');
  
  if (failures.length > 0) {
    const details = failures
      .map(f => `  User ${f.index}: ${f.reason?.message || f.reason}`)
      .join('\n');
    expect(failures, `${failures.length} flow(s) failed:\n${details}`).toHaveLength(0);
  }
}
```

### 3.7 Test User Fixtures (`tests/fixtures/test-users.json`)

Create 5 test users with email/password. If the registration form has additional required fields (like `displayName`), include those.

If admin roles exist, note that an admin user will be seeded separately in the setup script.

### 3.8 Test User Seeding (`tests/e2e/helpers/setup.{ext}`)

**Only create this if local emulators or a local database are available.**

Create a script that:
- Reads from `tests/fixtures/test-users.json`
- Creates users in the local auth system
- Creates corresponding database records with appropriate roles and any required fields (e.g., subscription status, onboarding completion) so tests can skip setup flows
- If admin role exists: seeds an admin user separately
- Skips users that already exist
- Logs what it creates
- Is runnable standalone: `node tests/e2e/helpers/setup.{ext}`

**Provider-specific seeding:**
- **Firebase**: Use `firebase-admin` SDK with emulator env vars
- **Supabase**: Use `@supabase/supabase-js` with local service URL
- **Prisma**: Use Prisma client to seed the DB directly
- **Custom**: Use the project's existing user creation logic or API endpoints

### 3.9 Auth Tests

**User login tests** (`tests/e2e/auth/user-login.spec.{ext}`):
- Read creds from `E2E_USER_EMAIL` / `E2E_USER_PASSWORD` env vars with defaults
- Test: user can log in and reach the main authenticated page
- Test: user can access key protected routes (detected in Phase 1)
- Test: user cannot access admin routes (if admin role exists)
- Test: user can log out
- Use `domcontentloaded` for page waits (avoid `networkidle` — live content like timers/websockets prevent it)

**Admin login tests** (`tests/e2e/auth/admin-login.spec.{ext}`) — only if admin role detected:
- Read creds from `E2E_ADMIN_EMAIL` / `E2E_ADMIN_PASSWORD` env vars with defaults
- Test: admin can log in
- Test: admin can access admin routes
- Test: admin can also access regular protected routes

**Invalid credentials tests** (`tests/e2e/auth/invalid-creds.spec.{ext}`):
- Test: shows error for wrong password
- Test: shows error for nonexistent email
- Test: blocks invalid email format (check HTML5 validation if applicable)
- Test: login form fields and submit button are present

### 3.10 Smoke Test (`tests/e2e/smoke/health.spec.{ext}`)

A minimal test **safe to run against production** — NO writes, NO account creation:
- Navigate to `/` (home page)
- Assert page loads with expected title (detected from `index.html` or layout)
- Assert no console errors (collect via `page.on('console')`)
- Check key elements render (detected from the homepage component):
  - Logo / brand name
  - Navigation links
  - Primary CTA
- Does NOT authenticate, write data, or modify state

### 3.11 Single-User Flow Tests (`tests/e2e/single/*.spec.{ext}`)

Create one spec file per flow definition. Each test imports the flow and runs it once in a standard Playwright page:

```javascript
// tests/e2e/single/browse.spec.{ext}
import { test } from '@playwright/test';
import { browseFlow } from '../flows/browse.flow.{ext}';

test('browse flow completes successfully', async ({ page }) => {
  await browseFlow(page);
});
```

Create a spec for every flow generated in section 3.6. These are your "does this walkthrough work" tests — run them to verify each journey is correct before using them in concurrent tests.

### 3.12 Multi-User Concurrent Tests (`tests/e2e/concurrent/multi-user.spec.{ext}`)

Create tests that import flow definitions and run them in parallel using separate browser contexts. Read `CONCURRENT_USERS` from the env config (default 5).

**Homogeneous concurrency** — N users running the same flow:

```javascript
import { test } from '@playwright/test';
import { dashboardFlow } from '../flows/dashboard.flow.{ext}';
import { assertAllFlowsSucceeded } from '../flows/helpers/concurrency.{ext}';
import { getConfig } from '../helpers/env.{ext}';
import testUsers from '../../fixtures/test-users.json';

const { concurrentUsers } = getConfig();

test('multiple users access dashboard simultaneously', async ({ browser }) => {
  const users = testUsers.slice(0, concurrentUsers);
  const results = await Promise.allSettled(
    users.map(async (user) => {
      const context = await browser.newContext();
      const page = await context.newPage();
      try {
        await dashboardFlow(page, user);
      } finally {
        await context.close();
      }
    })
  );
  assertAllFlowsSucceeded(results);
});
```

**Mixed traffic simulation** — different flows running concurrently (the realistic scenario):

```javascript
test('mixed traffic simulation', async ({ browser }) => {
  const newPage = async () => {
    const ctx = await browser.newContext();
    return { page: await ctx.newPage(), ctx };
  };

  const tasks = [
    ...Array(Math.ceil(concurrentUsers * 0.5)).fill(null).map(async () => {
      const { page, ctx } = await newPage();
      try { await browseFlow(page); } finally { await ctx.close(); }
    }),
    ...testUsers.slice(0, Math.ceil(concurrentUsers * 0.3)).map(async (user) => {
      const { page, ctx } = await newPage();
      try { await dashboardFlow(page, user); } finally { await ctx.close(); }
    }),
    ...testUsers.slice(0, Math.ceil(concurrentUsers * 0.2)).map(async (user) => {
      const { page, ctx } = await newPage();
      try { await registrationFlow(page, user); } finally { await ctx.close(); }
    }),
  ];

  const results = await Promise.allSettled(tasks);
  assertAllFlowsSucceeded(results);
});
```

**Key details:**
- Each flow gets its own browser context — full isolation
- `assertAllFlowsSucceeded` ensures no silent failures from `allSettled`
- Default 5 concurrent users locally (safe for laptops running emulators + dev server + N Chromium contexts). Scale via `CONCURRENT_USERS` env var in CI.
- Screenshots on failure per-user via Playwright's built-in failure handling
- Mixed traffic ratios (50/30/20) are a starting point — adjust based on the project's actual usage patterns

### 3.13 Package.json Scripts

Add scripts using the detected package manager and framework. Template:

```json
{
  "pretest:e2e": "lsof -ti:${TEST_PORT:-{test_port}} | xargs kill -9 2>/dev/null || true",
  "test:e2e": "{emulator_wrapper_start}node tests/e2e/helpers/setup.{ext} && npx playwright test --project=local{emulator_wrapper_end}",
  "test:e2e:quick": "npx playwright test tests/e2e/smoke/ tests/e2e/auth/invalid-creds.spec.{ext}",
  "test:e2e:prod": "TEST_TARGET=prod npx playwright test --project=smoke-only",
  "test:e2e:smoke": "npx playwright test tests/e2e/smoke/",
  "test:e2e:auth": "{emulator_wrapper_start}node tests/e2e/helpers/setup.{ext} && npx playwright test tests/e2e/auth/{emulator_wrapper_end}",
  "test:e2e:flows": "{emulator_wrapper_start}node tests/e2e/helpers/setup.{ext} && npx playwright test tests/e2e/single/{emulator_wrapper_end}",
  "test:e2e:concurrent": "{emulator_wrapper_start}node tests/e2e/helpers/setup.{ext} && npx playwright test tests/e2e/concurrent/{emulator_wrapper_end}",
  "test:e2e:headed": "{emulator_wrapper_start}node tests/e2e/helpers/setup.{ext} && npx playwright test --headed{emulator_wrapper_end}",
  "test:e2e:debug": "{emulator_wrapper_start}node tests/e2e/helpers/setup.{ext} && npx playwright test --headed --workers=1 --timeout=0{emulator_wrapper_end}",
  "test:e2e:codegen": "npx playwright codegen http://localhost:${TEST_PORT:-{test_port}}",
  "test:e2e:seed": "node tests/e2e/helpers/setup.{ext}",
  "test:e2e:logs": "{emulator_wrapper_start}node tests/e2e/helpers/setup.{ext} && E2E_LOG=1 npx playwright test{emulator_wrapper_end}"
}
```

Where `{emulator_wrapper_start/end}` is:
- **Firebase**: `firebase emulators:exec --only {services} '...'`
- **Supabase**: `supabase start && ... ; supabase stop` (or assume running)
- **Docker**: `docker-compose up -d && ... ; docker-compose down`
- **None**: just run the commands directly

**Only add staging scripts if user confirmed staging exists.**

**Design principles:**
- `test:e2e` is the default — starts everything needed, seeds data, runs all tests. No footguns.
- `test:e2e:flows` runs all single-user flow walkthroughs — verify each journey works before testing concurrency.
- `test:e2e:quick` runs only tests that don't need emulators/local services.
- `pretest:e2e` kills any process on the test port before starting.
- `reuseExistingServer: false` ensures Playwright starts its own dev server.

### 3.14 Gitignore

Add these entries to `.gitignore` (if not already present):

```
# E2E test artifacts
e2e-logs/
test-results/
playwright-report/
```

### 3.15 GitHub Actions Workflow (`.github/workflows/e2e.yml`)

Create a CI workflow that:
- Triggers on push and pull_request
- Installs dependencies and Playwright chromium
- Installs any required local services (Firebase CLI, Supabase CLI, etc.)
- Starts emulators/services, seeds data, runs tests
- Sets appropriate emulator env vars
- Uploads `test-results/` and `playwright-report/` as artifacts on failure

---

## Phase 4: Validate

1. Run `npx playwright test --list` to verify all tests are discovered
2. Show the user a summary of everything created
3. Suggest running `npm run test:e2e:quick` (or equivalent) as a first sanity check

---

## How to Approach This

1. **Do ALL of Phase 1 first** — read files, detect everything, build your mental model
2. **Ask Phase 2 questions** — present findings, proposed flows, get user input
3. **Scaffold everything in Phase 3** — use exact detected values, not placeholders. Generate flow helpers, flow definitions, single-user specs, and concurrent specs.
4. **Validate in Phase 4** — make sure it actually works
5. **Commit message**: `feat: add Playwright E2E testing framework with flow-based architecture`
