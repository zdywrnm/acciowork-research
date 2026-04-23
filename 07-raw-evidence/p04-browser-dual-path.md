# P0-4 Evidence: Browser Control Dual Path

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

## Code-level relay path

Bundled extension evidence:

- [manifest.json](/Applications/Accio.app/Contents/Resources/chrome-extension/accio-browser-relay/manifest.json:1)
- [background.js](/Applications/Accio.app/Contents/Resources/chrome-extension/accio-browser-relay/background.js:1)
- [constants.js](/Applications/Accio.app/Contents/Resources/chrome-extension/accio-browser-relay/lib/constants.js:1)

Key facts:

- extension name: `Accio Browser Relay`
- control port default: `9234`
- relay port: `controlPort + 2`, default `9236`
- permissions include `debugger`, `tabs`, `scripting`, `storage`, `notifications`, `<all_urls>`
- on install, the extension sets `relayEnabled=true`
- on startup, it attempts `connectAndAttach()` if relay is enabled

This establishes the intended preferred path:

`Accio app ⇄ local relay server ⇄ Chrome extension ⇄ chrome.debugger / CDP`

## Positive runtime evidence for fallback

Observed repeatedly in `sdk.log.1`:

```text
BrowserRelay: GET /status → extension: connected=false handshakeOk=false wsExists=false mismatch=false
DirectCDP: Chrome M144+ discovered at port 9222
```

Interpretation:

1. the app checks relay status first
2. when relay is unavailable, it falls back to raw Chrome discovery on `9222`
3. this is a real runtime path, not only dead code

## Temporary-profile relay attempt

Test performed:

1. launched a temporary Chrome profile under `/tmp/accio-relay-profile`
2. passed `--load-extension=/Applications/Accio.app/Contents/Resources/chrome-extension/accio-browser-relay`
3. observed Chrome process startup with the temporary profile
4. checked Accio logs after startup

Result:

- Accio still reported `connected=false`
- fallback to `DirectCDP` on `9222` remained active
- I did not obtain a positive `connected=true` relay sample in this round

## What this means

Defensible conclusion for now:

1. relay path is definitely implemented and bundled
2. raw `9222` fallback is positively observed in runtime
3. positive relay-handshake runtime proof remains incomplete in the current environment

This is enough to say the browser bridge is dual-path by design, but only the fallback side is fully runtime-validated so far.
