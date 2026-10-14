# `--add-dir <dir> "prompt"` silently eats your prompt

**Tool:** Claude Code CLI
**Version:** 2.1.285 (first seen on 2.1.x in July 2026)
**Date:** 2026-10-14

## Problem

Launching Claude Code with a directory and a starting prompt:

```bash
claude --add-dir /some/dir "Read BRIEF.md and do the task"
```

In interactive mode the session boots, the input box is empty, the agent sits
idle, and no transcript is ever created. Nothing errors. If you're launching
agents in panes or scripts, it looks exactly like a hung agent.

In print mode the same command fails loudly, which is the clue:

```
$ claude -p --add-dir /tmp "Reply with exactly: OK"
Error: Input must be provided either through stdin or as a prompt argument when using --print
```

## Root cause

`--add-dir` takes a variadic list (`<directories...>`), so the space form
consumes every following positional argument, including your prompt, as
another directory. Every other flag can be right and the prompt still never
arrives.

Verified with a single-flag A/B: adding `--add-dir <dir>` (space form) is the
only change needed to break it; `--agent`, `--permission-mode`, and `--model`
alone all submit fine.

## Fix

Use the equals form, which takes exactly one value:

```bash
claude --add-dir=/some/dir "Read BRIEF.md and do the task"
```

```
$ claude -p --add-dir=/tmp "Reply with exactly: OK"
OK
```

For several directories, repeat the flag: `--add-dir=/a --add-dir=/b`.

## Notes

- Any variadic flag placed directly before a positional prompt has the same
  trap. Putting the prompt first, or the flag last, also works, but the equals
  form is the one that can't regress when someone reorders arguments.
