# P0-5 Evidence: Sandbox and Command Gate Engine

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

Primary code source:

- [p02-out-main-index.js](/Users/a1-6/research/acciowork/07-raw-evidence/p02-out-main-index.js)

## Policy parser and serializer

Observed parser / serializer:

```js
function ih(e){ ... return {pattern:n,decision:o,justification:d,bypassSandbox:l?"true"===l[1]:void 0} }
function oh(e,t,a){return `prefix_rule(pattern=[${e.map(e=>`"${e}"`).join(", ")}], decision="${t}"${a?', bypassSandbox="true"':""})\n`}
```

This matches the on-disk `policy.jsonl` / rule format:

- prefix rule pattern array
- decision
- optional justification
- optional `bypassSandbox`

## Command decision engine

Observed runtime:

```js
function Ah(e,t,a){return"allow"}
...
checkCommand(e,t,a){
  const r=this.current(), s=ch(e)??lh(e)??[e], n=r.checkMultiple(s,e=>"allow")
  switch(n.decision){
    case "deny":
      return {type:"deny", reason:Rh(e,n)}
    case "ask":
      return "never"===t
        ? {type:"deny", reason:"approval required by policy, but AskForApproval is set to Never"}
        : {type:"needs_approval", reason:Mh(e,n), proposedAmendment:i??Oh(n.matchedRules)}
    case "allow":
      return {type:"skip", bypassSandbox:!0===n.bypassSandbox, proposedAmendment:Dh(n.matchedRules)}
  }
}
```

Interpretation:

- prefix rules resolve to `deny / ask / allow`
- an unmatched command falls through to `allow`
- `allow` does **not** necessarily mean “unsandboxed”; it means the approval gate is skipped

## Sandbox backends and scopes

Observed enums:

```js
Jl.None = "none"
Jl.MacosSeatbelt = "seatbelt"
Jl.LinuxSeccomp = "seccomp"
Jl.WindowsRestrictedToken = "windows_sandbox"

Ql.ReadOnly = "read_only"
Ql.ReadWriteCwd = "read_write_cwd"
Ql.DangerFullAccess = "danger_full_access"
```

Platform selection:

```js
selectInitial(e,t){
  return "forbid"===t
    ? Jl.None
    : "require"===t
      ? this.getPlatformSandbox()??Jl.None
      : e===Ql.DangerFullAccess
        ? Jl.None
        : this.getPlatformSandbox()??Jl.None
}
```

On macOS, the sandbox transform explicitly shells through:

```js
l=["/usr/bin/sandbox-exec", ...e]
```

This is hard evidence that Accio is using a real OS sandbox on macOS, not only soft allow/deny logic.

## File-write scope

Observed seatbelt policy generation:

```js
if(e===Ql.DangerFullAccess)
  return {fileWritePolicy:'(allow file-write* (regex #"^/"))', fileWriteParams:[]}
```

For `read_write_cwd`, writable roots are derived and expanded into scoped seatbelt rules instead of global write access.

Interpretation:

- `danger_full_access` = unrestricted writes across `/`
- `read_write_cwd` = writable roots only
- therefore the top-level permission picker is really switching sandbox breadth

## Bypass cases

Observed executor path:

```js
class ah{
  sandboxManager=new Np;
  async run(e,t,a){
    const r=e.getApprovalRequirement(t);
    if("deny"===r.type) throw new Error(r.reason);
    let s;
    s = "bypass_first"===e.sandboxModeForFirstAttempt(t) || "skip"===r.type && r.bypassSandbox
      ? Jl.None
      : this.sandboxManager.selectInitial(a.sandboxPolicy,e.sandboxPreference());
    ...
  }
}
```

Meaning:

- some tools or rules can intentionally bypass the sandbox on first attempt
- any policy rule with `bypassSandbox=true` also suppresses the OS sandbox

## Approval trace taxonomy

Observed embedded trace metadata:

- `tool.permission.asked`
- `tool.permission.approved`
- `tool.permission.denied`
- `tool.permission.auto_granted`

Descriptions inside the bundle explicitly say:

- `approved` includes UI `allow / allow_always`
- `denied` includes UI `denied / denied_always` plus policy auto-deny
- `auto_granted` is a pure policy/safety pass with no user approval step

## Bottom line

The shell/file execution path on macOS is:

`prefix-rule engine -> optional user approval -> macOS sandbox-exec seatbelt -> tool execution`

with separate escape hatches for `bypassSandbox` and `danger_full_access`.
