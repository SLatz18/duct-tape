# Deno Unix-socket server goes deaf but stays "healthy" on macOS

**Tool:** Deno (`Deno.listen({ transport: "unix" })`)
**Version:** Deno 2.9.7, macOS (aarch64)
**Date:** 2026-10-14

## Problem

A long-running Deno server that listens on a Unix socket stops answering.
Clients connect, then hang until their own timeout. But every liveness check
passes:

- the process is running and its pid file is valid
- the socket file exists and is bound
- `nc -U <socket>` connects (the connection is queued, never accepted)
- any other listeners in the same process (for example a WebSocket server) keep working

The log shows one rejection and then nothing, ever:

```
TypeError: Invalid argument (os error 22)
```

## Root cause

Two things combine:

1. **The accept loop dies on its first error.** With
   `for await (const conn of listener) { ... }`, a single rejected `accept()`
   ends the loop. On macOS, a client that connects and then closes before the
   server accepts it makes `accept()` reject with `Invalid argument (os error 22)`.
2. **A global error handler hides the death.** A
   `globalThis.addEventListener("unhandledrejection", e => e.preventDefault())`
   (common "don't crash the daemon" boilerplate) keeps the process alive after
   the loop is gone. So you get a half-dead process: up, bound, deaf.

It's a race. A client that closes synchronously right after connecting loses it
every time. A normal client loses it occasionally, more often when the server's
event loop is slow (CPU load). In our case the trigger was our own CLI's
"is it up?" check, which connected and immediately closed.

## Fix

Accept explicitly, log, and keep going:

```ts
const listener = Deno.listen({ transport: "unix", path });
while (true) {
  let conn: Deno.UnixConn;
  try {
    conn = await listener.accept();
  } catch (err) {
    if (err instanceof Deno.errors.BadResource || err instanceof Deno.errors.Interrupted) {
      return; // listener closed on purpose
    }
    console.error("accept error", err);
    await new Promise((r) => setTimeout(r, 50)); // brief backoff, then keep listening
    continue;
  }
  handle(conn); // don't await here, or one slow client blocks the rest
}
```

And make health checks real: send an actual request and wait for the reply.
"Can I connect?" is not "is it serving?". A connect-and-close probe is both a
false positive and, here, the trigger.

## Notes

- To reproduce: connect to the socket and close it with no delay, in a tight
  loop, from a synchronous client (Python `socket` works). The server stops
  accepting after the first loss.
- If you keep a global `unhandledrejection` handler, log inside it. A silent
  `preventDefault()` is what turns a crash you'd notice into an outage you won't.
