# P0-3 Evidence: Live Model Family Comparison

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

## Test setup

Three one-turn live prompts were sent from the same built-in `Accio` agent:

- `Standard` → Nexus family
- `Professional` → Orbit family
- `Qwen 3.6 Plus` → Drift family

Prompt pattern:

- `P03LIVE... respond exactly ACK...`

For comparison, only the main `round=0` request was used. Title-generation calls were excluded from the prompt-hash comparison.

## Routing summary

| UI choice | Family | Internal model code | Endpoint | Notes |
|---|---|---|---|---|
| `标准` | `Nexus` | `1Nexus-R3wF8qJ5vB6h` | `https://phoenix-gw.alibaba.com/api/adk/llm/generateContent?sg_k=<REDACTED>` | single main round |
| `专业` | `Orbit` | `1Orbit-I9eY4tK8bW1f` | `https://phoenix-gw.alibaba.com/api/adk/llm/generateContent?sg_k=<REDACTED>` | observed extra `round=1` follow-up after main round |
| `Qwen 3.6 Plus` | `Drift` | `1Drift-V4jL6mX3yE8d` | `https://phoenix-gw.alibaba.com/api/adk/llm/generateContent?sg_k=<REDACTED>` | single main round |

## Payload comparison

Main request comparison across the three families:

- `systemHash`: `ec578e2ee3c9a9fc5e23264380fb2915b3716b0a9e97cb75dcc9edd6f5376bfb`
- `systemLen`: `39464`
- `toolCount`: `32`
- `toolHash`: `4adffcef4e227ab64bc6d13ce1d2409ce4fce67da12416a21806fe824641963c`

Interpretation:

- the client-side main `systemInstruction` is identical across `Nexus / Orbit / Drift`
- the tool list emitted by the client is also identical
- model switching changed the `model` field, but not the client-emitted system prompt or tool inventory

## Representative outgoing body shape

The module-level transport interception produced this normalized request shape before network send:

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

Live desktop traces add the full system prompt, tool array, and trace metadata, but the endpoint family remains the same.

## Usage / quota fields

Observed usage fields in `adk.llm.response` / `llm.call` traces:

- `promptTokenCount`
- `candidatesTokenCount`
- `totalTokenCount`
- `thoughtsTokenCount`
- `cachedContentTokenCount`

Not observed in the captured LLM traces:

- explicit `rateLimit`
- explicit `remainingQuota`
- explicit `retry-after`

Separate product-level quota evidence exists outside the LLM response path, including:

- `GET /api/entitlement/quota`
- UI config payloads for “credits left” and exhaustion messages

Conclusion:

- quota UX exists
- but it does not appear to be returned in the captured LLM response payload shape
