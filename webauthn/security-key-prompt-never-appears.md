# Security Key prompt never appears on the Mac (WebAuthn timed out)

**Tool:** Cloud agent browser (Chromium) forwarding WebAuthn to a Mac desktop app
**Tool version:** unknown
**AI model:** Grok Bot
**Harness:** Grok Bot (desktop agent: cloud box Chromium + Grok Bot Mac app WebAuthn forwarding)
**Date:** 2026-10-15

## Problem

Signing into Okta (or a similar IdP) from a remote/cloud Chromium browser, the
Security Key / biometric MFA step fails with:

```
The operation either timed out or was not allowed
```

**Retry** does nothing. The Mac never shows a WebAuthn prompt, so the target
SaaS app behind SSO never gets past the IdP.

## Root cause

Two things have to be true at once. Either one is enough to fail the ceremony.

1. **The forwarder isn't armed.** WebAuthn forwarding needs a live arm window:
   about 20 minutes from active browser automation, or about 45 minutes from an
   explicit desktop handoff to the user. An open computer view alone may not
   arm it. An earlier handoff that is still sitting open may already be dead.
2. **The IdP challenge is stuck, or the gateway credential is missing.** Stale
   site data for that IdP host can leave a stuck MFA challenge, so Retry keeps
   replaying the same dead ceremony. Separately, the host↔box gateway
   credential is what carries the ceremony to the Mac. If it isn't present, the
   browser times out with no prompt on the Mac even when the arm window is live.

## Fix

Clear only that IdP host, start a fresh SSO, then re-arm. In order:

1. In the cloud agent browser, delete **only that IdP host's** site data
   (cookies and storage for the Okta/IdP origin). Do not wipe all browsing
   data. Open
   `chrome://settings/content/siteDetails?site=https%3A%2F%2F<idp-host>`
   (URL-encode the origin), click **Delete data**, and confirm. The dialog
   names only that site.
2. Start a **fresh** SSO sign-in from the target SaaS app so it redirects to
   the IdP. Don't resume the timed-out challenge.
3. Fill the username and password via the password manager / credential fill.
4. Issue a **new** desktop handoff. That re-arms forwarding for about 45
   minutes. An earlier open handoff may not be arming.
5. On the Security Key page, click **Retry** and tap the key when the Mac
   prompts.

If that still never prompts, fully quit and reopen the Mac desktop app that
forwards WebAuthn, then issue a new handoff and click **Retry** again.
Quitting the app alone does not re-arm a dead window.

## Notes

- Phone push and authenticator-app Verify may not be available for some users.
  Prefer a fresh SSO plus a new handoff that re-arms forwarding.
- This only clears that IdP origin. Other sites in the cloud browser stay
  signed in.
- If the Mac app is running, the handoff is new, and there is still no prompt,
  check that the host↔box gateway credential is present. Without it the
  ceremony never leaves the cloud browser.
