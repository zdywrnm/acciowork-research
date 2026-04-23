# P0-5 Evidence: `pairings-manager` Role

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

## Local storage state

Pairings directory on this machine:

- `/Users/a1-6/.accio/accounts/1758713785/pairings/`

Current state:

- directory exists
- no pairings files are present yet

## Code-level responsibilities

From the compiled main bundle, `pairingsManager` is referenced in:

- conversation list / conversation create flows
- IM display-name enrichment
- permission notification routing
- team-member verification

Representative bundle strings:

```text
channel.pairings
TEAM_CHANNEL_MEMBER_PAIRING_VERIFICATION_UNAVAILABLE
TEAM_CHANNEL_MEMBER_NOT_PAIRED
request_created
user_approved
user_removed
request_removed
request_updated
user_updated
```

## Observed behaviors

### 1. Team conversation gate

When a team conversation contains channel members, the create/update path checks pairings first:

```text
TEAM_CHANNEL_MEMBER_PAIRING_VERIFICATION_UNAVAILABLE
TEAM_CHANNEL_MEMBER_NOT_PAIRED
```

That means pairings are used as an eligibility check before a team channel conversation is allowed to proceed.

### 2. IM identity enrichment

`pairingsManager.getBySubject(...)` is used to resolve display names for Feishu and other IM identities when source-side names are opaque.

### 3. Permission approval routing

The bundle contains `notifyPermissionRequesterByIM(...)`, which sends a message like:

```text
🔐 该操作需要授权
⏳ 已收到你的请求……
请在 IM 侧回复授权，或等待桌面端处理。
```

This ties pairings directly to the remote approval flow for sensitive actions.

### 4. Pairing event stream

Observed event family:

- `channel.pairings`
- `request_created`
- `user_approved`
- `user_removed`
- `request_removed`
- `request_updated`
- `user_updated`

This looks like a durable IM/account pairing subsystem rather than a one-off browser attachment flow.

## Relationship to browser relay

Current conclusion:

- I found **no direct evidence** that `pairings-manager` is part of the Chrome relay handshake path
- browser relay evidence still lives under extension/CDP code and localhost relay state
- `pairings-manager` is best understood as `IM / channel identity + approval routing`

So for an open clone, pairing and browser-attachment should be modeled as separate subsystems unless later evidence proves coupling.
