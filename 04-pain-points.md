# Accio Work Pain Points

## Current pain points

1. Permission UX compresses multiple concepts into one surface. The UI shows `默认权限` vs `完全访问权限`, but the real runtime has separate `policy`, `approval result`, and `sandbox scope` layers. That makes it harder for users to predict what will actually happen.
2. Browser relay setup looks brittle. The bundle clearly prefers `relay extension`, but runtime success still fell back to raw `9222` CDP even after loading the bundled extension in an isolated Chrome profile.
3. The policy engine is safer than a pure allowlist, but still broad by default. Unmatched commands currently fall through to `allow`, so safety depends heavily on sandbox scope and whether a rule also carries `bypassSandbox=true`.
4. Pairings are product-critical but opaque. Team creation and IM-side permission approval depend on `pairings-manager`, yet the local on-disk state is sparse and the coupling to the rest of the product is not obvious from the UI.
5. Quota signaling is fragmented. The LLM traces show token usage, while user-visible “credits left / exhausted” messaging comes from separate entitlement/config payloads. That split makes debugging limits harder.

## Clone opportunity

1. Make the permission model explicit: show `policy decision`, `approval state`, and `sandbox scope` as separate concepts.
2. Treat browser control as a pluggable transport with clear health diagnostics instead of one opaque “browser connected” state.
3. Default to a tighter policy baseline, then surface explicit per-agent overrides.
4. Keep pairing/remote approval as an optional subsystem, not a hidden dependency for core local-agent workflows.
