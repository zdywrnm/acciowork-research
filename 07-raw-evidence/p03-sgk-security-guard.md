# P0-3 Evidence: `sg_k` and Security Guard

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

## Bundle-level evidence

Observed package paths inside `app.asar` / `app.asar.unpacked`:

- `node_modules/@phoenix-common/security-guard/dist/index.js`
- `node_modules/@phoenix-common/security-guard/dist/get-security-factors.js`
- `node_modules/@phoenix-common/security-guard/prebuild/darwin-arm64/security_guard.node`

This confirms the desktop app carries a native security-guard implementation locally.

## Transport-interceptor evidence from the main bundle

The compiled main bundle wires a transport interceptor that:

1. checks feature flag `sg_sign_llm`
2. calls `getSecurityFactorsForWebHeaders({ url: t.url })`
3. merges returned headers into the outgoing request
4. records the signing result through `onSecurityGuardResult(...)`

That is enough to show signing happens on the client, before the LLM request leaves the desktop app.

## Extracted security-guard helper

From the loaded `@phoenix-common/security-guard` module:

```js
function getSecurityFactorsForWebHeaders(params) {
  const urlStr = params.url?.trim() ?? '';
  const { api, requestUriForMd5 } = parseUrlForWebFactors(urlStr);
  const input = process.platform === 'win32'
    ? JSON.stringify({ appkey: appKey, urlInput: urlStr })
    : JSON.stringify({ appkey: appKey, api, data: md5Hex(requestUriForMd5) });
  raw = getSecurityFactorsForWeb(input);
}
```

And:

```js
function urlSign(appkey, input) {
  return addon.urlSign(appkey, input);
}
```

Interpretation:

- on macOS, the signer derives factors from the request URL, including an MD5 of the normalized request URI
- the native addon performs the actual signing work
- this is strong evidence that `sg_k` is client-generated, not first issued by the server

## Module-level transport interception

Using `@ali/accio-adk-ts` with a custom `transportInterceptor.beforeRequest`, I intercepted the final request before network send.

Observed request shape:

```json
{
  "url": "https://phoenix-gw.alibaba.com/api/adk/llm/generateContent?sg_k=<REDACTED>",
  "method": "POST",
  "headers": {
    "Content-Type": "application/json",
    "Accept": "text/event-stream",
    "Connection": "close"
  }
}
```

This proves `sg_k` is already attached before the HTTP request is emitted.

## Controlled comparison

With a fixed request body and fixed `requestId`:

- repeated calls with the same tenant produced the same `sg_k`
- switching tenant from `accio-agent` to `fake-tenant-qwen` produced the same `sg_k`

Interpretation:

- `sg_k` is deterministic for the normalized request in this harness
- tenant is not part of the final `/api/adk/llm/generateContent` wire format under this test
- signing is tied to the request URL and client-side signing inputs, not to an opaque server-issued nonce

## Final judgment

The most defensible reading is:

1. `sg_k` is generated on the client
2. generation depends on the outgoing URL / normalized request URI
3. the signing chain is implemented by a native security-guard addon bundled with the desktop app
