---
name: e2e-visual-audit
description: >
  Visually test recent git changes in the browser using Chrome DevTools MCP.
  Use when user wants to "e2e audit", "visual test recent changes", "test the UI",
  "check last commits in browser", "regression test my changes", "smoke test the UI",
  "QA my recent work", "visual QA", or invokes /e2e-audit. Analyzes N recent commits,
  maps changed files to UI routes, navigates each in Chrome, interacts with changed
  features, takes screenshots, checks console and network, and produces a structured
  issues report with organized screenshots.
compatibility: >
  Requires Chrome DevTools MCP server connected. Works with any web framework
  (SvelteKit, Next.js, React, Vue, Nuxt, etc). Requires a running dev server.
metadata:
  author: arch1904
  version: 1.0.0
  category: testing
  tags: [e2e, visual-testing, regression, chrome-devtools, qa]
---

# E2E Visual Audit

Analyze the last N git commits, map changed files to UI routes, then use Chrome DevTools MCP to navigate each affected route, interact with changed features, take screenshots, check console logs and network requests, and produce a structured issues report.

## Prerequisites

- **Chrome DevTools MCP** connected (the skill calls `list_pages`, `navigate_page`, `take_screenshot`, `take_snapshot`, `click`, `fill`, `list_console_messages`, `list_network_requests`, `resize_page`)
- **Dev server running** — detect the URL from project config (`package.json` scripts, vite/next/nuxt config) or ask the user
- **Authenticated session** — if the app requires login, the user must already be logged in within Chrome

If any prerequisite is missing, tell the user exactly what's needed and stop. Do not proceed with partial setup.

## Workflow Overview

```
1. Verify prerequisites (Chrome MCP, dev server, auth)
2. Analyze last N commits (git log + diff)
3. Map changed files → UI routes (framework-aware)
4. Present test plan → user approves
5. Create screenshot directory structure
6. Execute: navigate, interact, screenshot, check logs — per route
7. Compile report with linked screenshots
```

## Step 1: Verify Prerequisites

1. Call `list_pages` — if this fails, Chrome DevTools MCP is not connected. Tell the user to start it.
2. Detect dev server URL from project config. Check `package.json` `scripts.dev` for the port. If unclear, ask.
3. Call `navigate_page` to the dev server URL.
4. Call `take_snapshot` to verify the page loaded. Check the a11y tree for login forms or error states.
5. **If login is required**, follow the Auth Gate procedure below.
6. If all checks pass, proceed.

### Auth Gate — Handling Login Screens

If the snapshot shows a login form, sign-in page, or auth redirect at any point (initial load or mid-audit when navigating to a new route):

1. **Screenshot the login screen** so the user can see what you're seeing:
   ```
   take_screenshot(filePath: "{RUN_DIR}/_issues/auth-gate-{route-slug}.png")
   ```

2. **Present the user with options:**
   > I've hit a login screen at `{url}`. To continue the audit, I need an authenticated session. You have a few options:
   >
   > **Option A — Log in manually (recommended):** Switch to the Chrome window, log in, then tell me to continue. I'll re-navigate and pick up where I left off.
   >
   > **Option B — Provide test credentials:** Give me a test username/password and I'll fill the login form. Only do this with throwaway test accounts, never production credentials.
   >
   > **Option C — Skip authenticated routes:** I'll skip all routes that require auth and only test public pages. I'll note skipped routes in the report.
   >
   > **Option D — Provide a session cookie or auth token:** If you can give me a cookie value or bearer token, I can inject it via `evaluate_script`.

3. **Based on user's choice:**
   - **Option A:** Wait for the user to say "continue" or "done". Then call `take_snapshot` to verify login succeeded. If still on login screen, let them know and ask again.
   - **Option B:** Use `fill_form` or `fill` to enter credentials, then `click` the submit button. After submission, `take_snapshot` to verify login succeeded. If it fails (wrong credentials, 2FA prompt, CAPTCHA), report the failure and fall back to asking for Option A.
   - **Option C:** Add all auth-required routes to a "skipped" list. Continue with public routes only. Note all skipped routes prominently in the report.
   - **Option D:** Use `evaluate_script` to set the cookie or localStorage token:
     ```
     evaluate_script(function: "() => { document.cookie = '{name}={value}; path=/'; }")
     // or for localStorage:
     evaluate_script(function: "() => { localStorage.setItem('{key}', '{value}'); }")
     ```
     Then call `navigate_page` to reload and `take_snapshot` to verify auth.

4. **If auth expires mid-audit** (navigating to a new route shows login again): Pause testing, screenshot the state, and re-run the Auth Gate procedure. Do not silently skip routes — the user should always know when auth blocks testing.

## Step 2: Analyze Recent Commits

Default N=5. User can override via argument (e.g., `/e2e-audit 3`).

```bash
git log --oneline -N
git diff --name-only HEAD~N..HEAD
git diff --stat HEAD~N..HEAD
```

**Categorize** each changed file:
- **Route/page files** → direct UI pages to test
- **Component files** → grep for imports to find which routes use them
- **Service/state/store files** → trace to UI via component imports
- **API routes** → test indirectly via the UI features that call them
- **Style/config/tailwind files** → visual regression candidates across multiple routes
- **Non-UI files** (migrations, scripts, docs) → skip, note in report as "not testable in UI"

**If the diff is large (30+ files):** Group by feature area and focus on the most impactful routes. Ask the user which areas to prioritize rather than testing everything.

## Step 3: Map Files to Testable Routes

Detect the framework from `package.json` dependencies and apply its routing convention:

| Framework | Route directory | URL mapping |
|---|---|---|
| SvelteKit | `src/routes/` | Directory = URL path, `+page.svelte` = page |
| Next.js (app) | `app/` or `src/app/` | Directory = URL path, `page.tsx` = page |
| Next.js (pages) | `pages/` or `src/pages/` | File = URL path |
| Nuxt | `pages/` | File = URL path |
| React Router | Check route config file | Read router setup |
| Vue Router | Check route config file | Read router setup |
| Other | Ask user | User provides URL mapping |

**Component → route resolution:** Grep for the component name across route files. If used in multiple routes, pick 2-3 most representative ones. For shared primitives (button, input, card), test on the most complex page that uses them.

**Dynamic route segments** (`[id]`, `[slug]`, `{param}`): Navigate to the app and extract real parameter values from sidebar links, lists, or breadcrumbs. If no values are visible, ask the user.

Present the route map to the user before proceeding. Ask if routes are missing or should be skipped.

## Step 4: Build and Present Test Plan

For each affected route, present a plan like:

```
Route: /dashboard/settings
Changed: SettingsForm.svelte (added validation), theme-toggle.ts (dark mode fix)
Tests:
  1. Navigate → screenshot initial state
  2. Check console + network
  3. Fill settings form with test data → screenshot
  4. Toggle theme → screenshot
  5. Responsive check (tablet + mobile)
  6. Re-check console + network after interactions
```

Present the full plan. Ask the user to approve, modify, or skip routes before executing.

## Step 5: Create Screenshot Directory

Generate a timestamped run folder. This prevents audits from overwriting each other and keeps results browsable.

```bash
RUN_DIR="./test-output/e2e-audit/YYYY-MM-DD_HH-MM"   # Use actual timestamp
mkdir -p "$RUN_DIR/_issues"
# Create route subdirectories as you go in Step 6
```

Directory structure:
```
./test-output/e2e-audit/2026-03-04_14-30/
├── settings/
│   ├── 01-initial-load.png
│   ├── 02-after-fill-form.png
│   └── 03-tablet-768px.png
├── checkout/
│   ├── 01-initial-load.png
│   └── 02-after-click-pay.png
├── _issues/
│   └── checkout--uncaught-null-ref.png
└── REPORT.md
```

See `references/report-template.md` for naming conventions and the full report template.

## Step 6: Execute Tests

For each route in the approved plan:

### 6a. Navigate and Capture Initial State
```
mkdir -p {RUN_DIR}/{route-slug}/
navigate_page(url: "{devServerUrl}/{route}")
```
Wait for the page to settle. Then:
```
take_screenshot(filePath: "{RUN_DIR}/{route-slug}/01-initial-load.png", fullPage: true)
take_snapshot()
```
The snapshot provides the a11y tree with element `uid`s you need for interaction.

### 6b. Check Console and Network
```
list_console_messages(types: ["error", "warn"])
list_network_requests(resourceTypes: ["xhr", "fetch"])
```

**Filtering noise:** Ignore known framework dev-mode warnings (e.g., Svelte "hydration" warnings in dev, React strict mode double-renders, Next.js fast-refresh messages). Flag genuine errors — uncaught exceptions, failed API calls, undefined references.

If a genuine error is found, capture an issue screenshot:
```
take_screenshot(filePath: "{RUN_DIR}/_issues/{route-slug}--{error-description}.png")
```

### 6c. Interact with Changed Features

Read the diff to understand *what* changed, then interact accordingly:

| Change type | How to test |
|---|---|
| Button added/modified | `click(uid)` → verify response in snapshot |
| Form logic changed | `fill_form` with test data → submit → check result |
| List/table rendering | Verify data renders, try sorting or filtering |
| Modal/dialog | Open it, verify content, close it |
| Navigation/routing | Click links, verify URL changes |
| Text/copy changes | Verify correct text in snapshot |
| Styling/layout | Visual comparison in screenshot |

After each interaction, take a sequentially numbered screenshot and re-check console:
```
take_screenshot(filePath: "{RUN_DIR}/{route-slug}/02-after-{action}.png", fullPage: true)
list_console_messages(types: ["error", "warn"])
```

### 6d. Responsive Check

Run this if any layout, style, or CSS changes were detected in the diff:
```
resize_page(width: 768, height: 1024)    # Tablet
take_screenshot(filePath: "{RUN_DIR}/{route-slug}/{NN}-tablet-768px.png", fullPage: true)
resize_page(width: 375, height: 812)     # Mobile
take_screenshot(filePath: "{RUN_DIR}/{route-slug}/{NN}-mobile-375px.png", fullPage: true)
resize_page(width: 1440, height: 900)    # Restore desktop
```

Look for: overflow, text clipping, overlapping elements, broken layouts, touch target sizes.

## Step 7: Compile Report

Read `references/report-template.md` for the full template. Create `{RUN_DIR}/REPORT.md` and present a summary in chat.

Key requirements:
- All screenshot paths must be **relative** to the run directory (so clickable from the markdown file)
- Include a **Screenshot Index** table listing every screenshot with its route and description
- Classify every issue by severity (Critical / Warning / Info)
- Link each issue back to the **specific commit** and **changed file** that caused it
- If no issues were found, say so explicitly — a clean audit is valuable information

At the end, tell the user:
```
open {RUN_DIR}
```
So they can browse screenshots in their file manager.

## Issue Classification

| Severity | What qualifies | Examples |
|---|---|---|
| **CRITICAL** | Functionality is broken | JS exceptions in console, blank/crashed page, 500 errors, missing UI elements, uncaught errors |
| **WARNING** | UX is degraded but functional | Visual misalignment, overflow/clipping, slow requests (>3s), elements not responding to clicks, layout shifts, deprecation warnings |
| **INFO** | Minor observations | Slight spacing differences, dev-mode console logs, slow but successful loads, cosmetic nitpicks |

## Troubleshooting

### Chrome DevTools MCP not connected
`list_pages` fails or returns no pages. Tell the user to start Chrome DevTools MCP and verify it appears in their MCP server list. Common fix: restart the MCP server, ensure Chrome is open.

### Page loads but shows login screen
Follow the **Auth Gate** procedure in Step 1. Present all four options to the user (manual login, test credentials, skip auth routes, inject token). Do not silently skip the route or abort the audit — always give the user a choice.

### Route returns 404
The route may require dynamic parameters that don't exist in the current DB state. Mark it as "requires setup — no test data available" in the report. Ask the user if they can navigate there manually.

### Console is extremely noisy (100+ warnings)
Some frameworks emit many dev-mode warnings. Filter by focusing on `error` type messages first. For warnings, look for patterns — if the same warning repeats 50 times, note it once with the count. Do not screenshot every warning.

### Page takes too long to load
If `navigate_page` hangs or the snapshot shows a loading spinner, wait a few seconds and retry `take_snapshot`. If still loading after 10 seconds, screenshot the loading state, note it as a WARNING ("slow page load"), and move on.

### Diff is enormous (50+ files)
Do not try to test everything. Group files by feature, present the top 5-10 most impactful routes, and ask the user which to prioritize. Note skipped routes in the report.

### MCP disconnects mid-audit
Save whatever report data you have. List completed routes and remaining routes. The user can re-run `/e2e-audit` to continue — previous run folders are preserved (timestamped).

## Rules

These exist to keep audits consistent and safe:

- **Read-only** — never modify source code during an audit. The goal is to observe, not fix. If you find an issue, report it; the user decides what to fix.
- **User approval before execution** — always present the test plan first. The user may have routes they know are broken, or areas they don't want tested.
- **Screenshot everything** — when in doubt, take a screenshot. It's cheap and provides evidence. Use `fullPage: true` by default to catch overflow issues below the fold.
- **Sequence screenshots** — number them `01-`, `02-`, `03-` within each route folder so they sort chronologically when browsing.
- **Issue screenshots in `_issues/`** — prefix with route slug for traceability (e.g., `settings--form-validation-crash.png`).
- **Diff vs. reality** — compare what the diff says should have changed with what you actually see. If a commit claims to "fix overflow" but overflow is still visible, that's a CRITICAL finding.
