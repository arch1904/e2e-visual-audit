# E2E Visual Audit

An agent skill that visually tests your recent git changes in the browser using Chrome DevTools MCP.

Analyzes the last N commits, maps changed files to UI routes, navigates each route, interacts with changed features, takes screenshots, checks console logs and network requests, and produces a structured issues report.

Works with any web framework (SvelteKit, Next.js, React, Vue, Nuxt, etc).

## Install

Pick whichever method matches your setup:

### npx skills CLI (Claude Code, Codex, Cursor, 35+ agents)

```bash
npx skills add arch1904/e2e-visual-audit
```

### Claude Code — plugin marketplace

```
/plugin marketplace add arch1904/e2e-visual-audit
/plugin install e2e-visual-audit
```

### Claude Code — manual

```bash
git clone https://github.com/arch1904/e2e-visual-audit.git
cp -r e2e-visual-audit/skills/e2e-visual-audit ~/.claude/skills/
```

## Usage

```
/e2e-audit        # Test last 5 commits (default)
/e2e-audit 3      # Test last 3 commits
```

No manual setup required — the skill handles prerequisites automatically:

- **Chrome DevTools MCP** — guides you through install if not connected
- **Dev server** — runs a fresh build and starts the dev server automatically
- **Authentication** — interactive Auth Gate if login is needed

## What It Does

1. **Verifies prerequisites** — checks Chrome DevTools MCP, builds the project, starts the dev server
2. **Analyzes recent commits** — `git diff HEAD~N..HEAD` to find all changed files
3. **Maps files to routes** — auto-detects your framework (SvelteKit, Next.js, Nuxt, React Router, Vue Router) and maps changed files to testable UI routes
4. **Presents a test plan** — shows you exactly what will be tested, you approve before execution
5. **Executes tests per route** — navigates, takes screenshots, clicks/fills changed elements, checks console errors + network failures
6. **Responsive checks** — tablet (768px) and mobile (375px) screenshots for layout changes
7. **Compiles a report** — severity-ranked issues with linked screenshots in `REPORT.md`

## Automatic Setup

### Chrome DevTools MCP

If not installed, the skill provides the exact install command:

```bash
npx @anthropic-ai/claude-code mcp add chrome-devtools -- npx @anthropic-ai/chrome-devtools-mcp@latest
```

### Dev Server

The skill detects your package manager (`pnpm`, `yarn`, `bun`, `npm`), runs a fresh build, and starts the dev server in the background. If a dev server is already running, it skips this step.

### Authentication

If a login screen is encountered (at startup or mid-audit), the skill pauses and presents four options:

| Option | Description |
|--------|-------------|
| **Manual login** | Switch to Chrome, log in, tell the agent to continue |
| **Test credentials** | Provide throwaway credentials for the agent to fill |
| **Skip auth routes** | Test only public pages, note skipped routes in report |
| **Inject token** | Provide a session cookie or auth token to inject via JS |

## Screenshot Organization

Each run creates a timestamped folder that never overwrites previous runs:

```
./test-output/e2e-audit/2026-03-04_14-30/
├── dashboard/
│   ├── 01-initial-load.png
│   ├── 02-after-click-save.png
│   ├── 03-tablet-768px.png
│   └── 04-mobile-375px.png
├── settings/
│   ├── 01-initial-load.png
│   └── 02-after-toggle-theme.png
├── _issues/
│   ├── dashboard--uncaught-null-ref.png
│   └── settings--overflow-clipping.png
└── REPORT.md
```

## Issue Severity

| Level | Criteria |
|-------|----------|
| **CRITICAL** | JS errors, broken rendering, 500s, missing UI elements, uncaught exceptions |
| **WARNING** | Visual misalignment, overflow, slow requests (>3s), unresponsive elements, layout shifts |
| **INFO** | Minor spacing, dev-mode logs, cosmetic observations |

## Troubleshooting

The skill handles common issues automatically, including:

- Chrome DevTools MCP not connected → setup instructions
- Login screens → interactive Auth Gate
- Noisy console (100+ warnings) → filters dev-mode noise, focuses on errors
- Slow page loads → retries, screenshots loading state, moves on
- Large diffs (50+ files) → groups by feature, asks you to prioritize
- MCP disconnect mid-audit → saves partial report, lists remaining routes

## License

MIT
