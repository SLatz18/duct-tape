# `muse login` dies with "device flow transport error" behind proxies

**Tool:** Muse Code CLI
**Version:** 1.4.7 (1.4.7-R6101.1), likely others
**Date:** 2026-10-13

## Problem

`muse login` fails during the device-code flow:

```
login failed: device flow transport error: error sending request
```

The device-code request succeeds and the browser approval works, but the
CLI's polling loop dies at random — anywhere from 30 seconds to 4 minutes,
across 8 attempts. Each approval is orphaned when the CLI dies.

## Root cause

The `muse` binary's own HTTP client can't hold the polling connection in
proxied environments. This is **not** the network, the proxy, or the account:
50 sequential curl requests to `auth.meta.com` over 4+ minutes all returned
200 while the CLI kept dying. The failure is inside the binary's HTTP stack.

## Fix

Do the [RFC 8628](https://www.rfc-editor.org/rfc/rfc8628.html) device flow
manually with curl, mint an API key, and store it. Endpoints were
reverse-engineered by the [CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI/pull/5502)
project.

**1. Start the device flow** (trailing slashes are required — without them it's a 404):

```bash
curl -s -X POST "https://auth.meta.com/oidc/device/authorization/" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=1031625952748946"
```

Response includes `device_code`, `user_code`, and `verification_uri`
(e.g. `https://auth.meta.com/oauth/device/?code=XXXX-XXXX`). Codes expire in
600 seconds; poll every 5.

**2. Approve** the user code in your browser at the verification URI.

**3. Poll the token endpoint** until it returns an access token
(`authorization_pending` means keep waiting; `expired_token` and
`access_denied` are terminal):

```bash
curl -s -X POST "https://auth.meta.com/oidc/device/token/" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=urn:ietf:params:oauth:grant-type:device_code&device_code=<device_code>&client_id=1031625952748946"
```

Response: `{"access_token":"dca:...","token_type":"Bearer"}`.

**4. Mint an LLM API key** from the device token:

```bash
curl -s -X POST "https://api.meta.ai/muse-code/key" \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" -d '{}'
```

Response includes `api_key` (format `LLM|<id>|<secret>`), `base_url`
(`https://api.meta.ai/v1`), and subscription info. The key is tied to your
Muse Code subscription tier.

**5. Store it** (`TBH_CREDENTIAL_BACKEND=file` dodges a separate Linux
keychain-write bug; credentials land in `~/.config/muse/auth.json`, mode 0600):

```bash
export TBH_CREDENTIAL_BACKEND=file
echo '<api_key>' | muse auth set --provider meta --api-key-stdin
```

**6. Verify:**

```bash
muse exec "Reply with just the word: hello"
```

## Notes

- Never print or log the `device_code`, `access_token`, or `api_key`.
- A bug report was filed with the Muse team; this workaround stands until the
  CLI's HTTP client is fixed.
- Related: CLIProxyAPI issue [#5777](https://github.com/router-for-me/CLIProxyAPI/issues/5777)
  documents the same endpoints.
