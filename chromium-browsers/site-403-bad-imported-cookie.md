# One site returns 403 on every page in one browser (bad stored cookie)

**Tool:** Chromium-based browsers (seen in Aside, a Chromium 153 agent browser)
**Version:** Aside 1.0.928 / Chromium 153
**Date:** 2026-10-14

## Problem

One internal site (a self-hosted GitLab behind nginx) returned a bare nginx
`403 Forbidden` for every page in one browser, including the sign-in page,
while the same URLs worked in other browsers and from curl on the same machine.

## Root cause

A bad cookie stored for that site, most likely carried over by a profile import
from another browser. The front-end proxy rejected any request that carried it,
before the app ever saw the request.

How it was isolated, ruling out the usual suspects:

- **Not the network:** the browser and curl had the same public IP.
- **Not the user agent or headers:** replaying the browser's exact headers and
  UA with curl returned 200 every time.
- **Cookies:** from the broken page's own console:

```js
(await fetch(location.href, { credentials: "omit" })).status    // 200
(await fetch(location.href, { credentials: "include" })).status // 403
```

`document.cookie` only showed a few small cookies; the offending one is
HttpOnly, so page scripts can't see it.

## Fix

Delete that one site's stored data, nothing else:

1. Open `chrome://settings/content/siteDetails?site=https%3A%2F%2F<your-host>`
   (URL-encode the origin).
2. Click **Delete data** and confirm. The dialog names only that site.
3. Reload. You'll be signed out of that site and get the normal sign-in flow.

## Notes

- This only clears that origin's cookies and storage. It doesn't touch other
  sites or the rest of the profile.
- If you just imported a profile and several work sites act up the same way,
  check each with the `credentials: "omit"` test before clearing.
