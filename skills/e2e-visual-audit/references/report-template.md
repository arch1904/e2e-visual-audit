# Report Template

Use this template for the `{RUN_DIR}/REPORT.md` file. All screenshot paths must be relative to the run directory so users can click through from the report.

```markdown
# E2E Visual Audit Report

**Date**: {date}
**Project**: {project name from package.json}
**Commits**: {oldest_hash}..{newest_hash} ({N} commits)
**Dev server**: {url}
**Run folder**: `{run-folder-path}`

## Summary

| Metric | Count |
|--------|-------|
| Routes tested | X |
| Screenshots taken | Y |
| Critical issues | A |
| Warnings | B |
| Info | C |

## Screenshot Index

| Route | File | Description |
|-------|------|-------------|
| {route-slug} | [{NN}-{name}.png](./{route-slug}/{NN}-{name}.png) | {what it shows} |

## Issues

### CRITICAL — Broken functionality, errors, crashes

#### {Issue title}
- **Route**: /path/to/route
- **Screenshot**: [{filename}](./{route-slug}/{filename})
- **Issue screenshot**: [{filename}](./_issues/{filename})
- **Console error**: `{error message}`
- **Commit**: {hash} — "{message}"
- **Changed file**: {path/to/file}
- **Impact**: {what is broken and why it matters}

### WARNING — Visual regressions, UX problems

#### {Issue title}
- **Route**: /path/to/route
- **Screenshot**: [{filename}](./{route-slug}/{filename})
- **Description**: {what looks wrong}
- **Commit**: {hash} — "{message}"

### INFO — Minor observations

- {observation} — [{screenshot}](./{route-slug}/{filename})

## Route Details

### {/path/to/route}
- **Status**: PASS / FAIL
- **Screenshots**: {list with relative links}
- **Console errors**: {count} — {summary if any}
- **Network failures**: {count} — {summary if any}
- **Interactions tested**: {what was clicked/filled/navigated}
- **Notes**: {observations}

## Console Log Summary

### {route-slug}
| Level | Message | Count |
|-------|---------|-------|
| error | {message} | {N} |
| warn  | {message} | {N} |

### {route-slug}
(repeat per route)

## Network Summary

### Failed Requests
| Route | URL | Status | Type |
|-------|-----|--------|------|
| {route-slug} | {url} | {status} | {xhr/fetch} |

### Slow Requests (> 3s)
| Route | URL | Duration | Type |
|-------|-----|----------|------|
| {route-slug} | {url} | {ms} | {xhr/fetch} |
```

## Screenshot Naming Conventions

- **Run folder**: `{YYYY-MM-DD_HH-MM}` — date and time of audit start
- **Route subfolders**: Slugified route name (e.g., `instant-agent`, `design-system-brand-book`, `settings-general`)
- **Screenshot files**: `{NN}-{descriptive-name}.png` — zero-padded sequence number + description of what it shows
- **Issue screenshots**: `_issues/{route-slug}--{issue-description}.png` — route prefix for traceability

### Filename Examples

Good filenames:
- `01-initial-load.png`
- `02-after-click-submit-btn.png`
- `03-form-validation-error-state.png`
- `04-tablet-768px.png`
- `05-mobile-375px.png`
- `_issues/checkout--uncaught-null-ref-error.png`

Bad filenames:
- `screenshot1.png` (no description)
- `02-action.png` (too vague)
- `test.png` (no sequence, no context)
