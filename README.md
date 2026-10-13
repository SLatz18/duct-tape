# duct-tape 🩹

Fixes that hold things together. Workarounds, reverse-engineered endpoints,
and hard-won solutions for dev tools — documented so the next person doesn't
have to bleed for them.

## Index

| Tool | Problem | Fix |
|------|---------|-----|
| [Muse Code](muse-code/muse-login-device-flow.md) | `muse login` dies with `device flow transport error` behind proxies | Manual device flow with curl + API key mint |

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
