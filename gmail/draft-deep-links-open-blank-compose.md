# Gmail draft deep links open a blank compose window

**Tool:** Gmail web
**Version:** n/a (web, observed 2026-10)
**Date:** 2026-10-14

## Problem

You create a Gmail draft through the API or an integration and want to hand the
user a link that opens it. Links of the form:

```
https://mail.google.com/mail/u/0/#drafts?compose=<draft id>      (e.g. r123…)
https://mail.google.com/mail/u/0/#drafts?compose=<message id>    (hex, e.g. 1a12…)
```

both open the Drafts folder plus an empty "New Message" window. The draft you
wanted isn't opened.

## Root cause

Not confirmed. Gmail's `compose=` parameter doesn't resolve either ID that the
Gmail API returns for a draft, and falls back to a new blank compose.

## Fix

There's no working deep link that we found. Link to the Drafts folder and name
the subject:

```
https://mail.google.com/mail/u/0/#drafts
```

"Top of Drafts, subject: <subject>."

## Notes

- Close the blank compose window without typing in it. It can autosave as a junk
  "(no subject)" draft.
- Tested with both ID formats returned by the Gmail API on 2026-10-09.
