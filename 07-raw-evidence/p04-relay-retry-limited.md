# P0-4.5 Evidence: Limited Relay Retry

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

## Scope

This was a bounded retry only, per research limit:

- one isolated Chrome launch
- explicit remote-debugging port
- bundled relay extension loaded
- no more than a short validation window

## Test launch

Temporary Chrome profile launch used:

```text
/Applications/Google Chrome.app/Contents/MacOS/Google Chrome
  --user-data-dir=/tmp/accio-relay-rdp
  --load-extension=/Applications/Accio.app/Contents/Resources/chrome-extension/accio-browser-relay
  --remote-debugging-port=9222
```

Observed listeners during the test:

- Chrome: `127.0.0.1:9222`
- Accio relay/server side: `127.0.0.1:9236`

## Runtime log result

Observed Accio log state remained:

```text
BrowserRelay GET /status → extension: connected=false handshakeOk=false wsExists=false mismatch=false
DirectCDP Chrome M144+ discovered at port 9222
```

## Interpretation

What this proves:

1. loading the relay extension and enabling `--remote-debugging-port` is not sufficient by itself to complete the relay handshake in this environment
2. the `DirectCDP` fallback remains the positively verified runtime path
3. the missing step is likely product-side attachment / pairing / state coordination rather than simple port exposure

## Research disposition

This retry is being marked as:

- `limited attempt completed`
- `positive relay handshake still unverified`
- `safe to archive unless later product-side state becomes available`
