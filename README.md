# duct-tape 🩹

Fixes that hold things together. Workarounds, reverse-engineered endpoints,
and hard-won solutions for dev tools — documented so the next person doesn't
have to bleed for them.

## Index

| Tool | Problem | Fix |
|------|---------|-----|
| [Muse Code](muse-code/muse-login-device-flow.md) | `muse login` dies with `device flow transport error` behind proxies | Manual device flow with curl + API key mint |
| [Claude Code](claude-code/add-dir-swallows-prompt.md) | `--add-dir <dir> "prompt"` boots idle with an empty prompt | Use the equals form: `--add-dir=<dir>` |
| [Deno](deno/accept-loop-dies-silently-macos.md) | Unix-socket server goes deaf but passes every liveness check (macOS) | Explicit `accept()` loop that logs and continues; real health checks |
| [Chromium browsers](chromium-browsers/site-403-bad-imported-cookie.md) | One site 403s on every page in one browser, curl works | Delete that site's data via `chrome://settings/content/siteDetails` |
| [Gmail](gmail/draft-deep-links-open-blank-compose.md) | `#drafts?compose=<id>` opens a blank compose, not the draft | Link to `#drafts` and name the subject |

## Contributing

One markdown file per fix, in a folder named after the tool. Format:

```markdown
# <short problem title>

**Tool:** ...
**Version:** ...
**Date:** ...

## Problem
What breaks, and the exact error.

## Root cause
What you actually figured out (not guesses).

## Fix
Step-by-step commands.

## Notes
Anything else — credits, related issues, caveats.
```

PRs welcome. If it held your setup together, it belongs here.
