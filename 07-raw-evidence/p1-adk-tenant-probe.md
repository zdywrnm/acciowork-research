# P1 Quick Check: `ADK_TENANT_*` Usage and Probe

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

## Code paths found

The compiled main bundle reads these environment variables:

- `ADK_TENANT`
- `ADK_TENANT_OPENAI`
- `ADK_TENANT_GEMINI`
- `ADK_TENANT_CLAUDE`
- `ADK_TENANT_QWEN`
- `ADK_BASE_URL`
- `ADK_IAI_TAG`

Observed pattern:

- a default tenant such as `accio-agent`
- optional `tenantOverrides` map per provider family
- values passed into the local LLM provider config

This means the env vars are active runtime inputs.

## Synthetic module-level probe

To avoid restarting the full desktop app, I ran a module-level probe against:

- `@ali/accio-adk-ts`
- `AccioLlm`
- `transportInterceptor.beforeRequest`

### Probe A: request construction

`createAccioLlmRequest(...)` preserves the tenant in the in-memory request object:

```json
{
  "tenant": "accio-agent"
}
```

vs.

```json
{
  "tenant": "fake-tenant-qwen"
}
```

### Probe B: pre-network HTTP interception

The final intercepted HTTP request to `/api/adk/llm/generateContent` did **not** expose tenant in:

- URL path or query
- headers
- JSON body

With a fixed request and fixed request ID:

- normalized body was identical
- `sg_k` was identical

Representative normalized body:

```json
{
  "model": "1Drift-V4jL6mX3yE8d",
  "contents": [
    {
      "role": "user",
      "parts": [
        {
          "text": "ping"
        }
      ]
    }
  ],
  "system_instruction": "You are test"
}
```

## Interpretation

What this does prove:

1. `ADK_TENANT_*` is real code, not dead code
2. tenant can influence the in-memory request object or higher-level provider wiring

What this does **not** prove:

1. that tenant is sent directly to the public `/api/adk/llm/generateContent` wire payload
2. that Accio exposes a user-facing BYOK mode

Current judgment:

- this looks more like an internal routing / override hook than a straightforward BYOK backdoor
- a full hot-relaunch test of the desktop app with fake tenant env vars was intentionally not done in this pass
