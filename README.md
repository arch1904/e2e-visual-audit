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

## Prerequisites

- Dev server running (e.g. `localhost:5173`, `localhost:3000`)
- Chrome open with [Chrome DevTools MCP](https://github.com/nichochar/chrome-devtools-mcp) connected
- Logged into the app (if auth required)

## Usage

```
/e2e-audit        # Test last 5 commits (default)
/e2e-audit 3      # Test last 3 commits
```

## What It Does

1. Analyzes `git diff HEAD~N..HEAD` to find all changed files
2. Auto-detects your framework and maps files to UI routes
3. Presents a test plan for your approval
4. For each route: navigates, screenshots, clicks/fills changed elements, checks console errors + network failures
5. Responsive checks (tablet/mobile) for layout changes
6. Outputs a severity-ranked report with organized screenshots

## Screenshot Organization

Each run creates a timestamped folder:

```
./test-output/e2e-audit/2026-03-04_14-30/
├── instant-agent/
│   ├── 01-initial-load.png
│   ├── 02-after-send-message.png
│   ├── 03-tablet-768px.png
│   └── 04-mobile-375px.png
├── settings/
│   ├── 01-initial-load.png
│   └── 02-after-toggle-theme.png
├── _issues/
│   ├── instant-agent--console-error-null-ref.png
│   └── settings--overflow-clipping.png
└── REPORT.md
```

## Issue Severity

| Level | Criteria |
|-------|----------|
| **CRITICAL** | JS errors, broken rendering, 500s, missing UI elements |
| **WARNING** | Visual misalignment, overflow, slow requests (>3s), unresponsive elements |
| **INFO** | Minor spacing, dev-mode logs, cosmetic observations |

## License

MIT
