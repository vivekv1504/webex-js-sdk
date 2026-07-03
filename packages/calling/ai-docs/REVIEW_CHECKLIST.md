<!-- ───────────────────────────────
  Template:     Review-Check Catalog
  Template-ID:  review-checklist
  Generates:    ai-docs/REVIEW_CHECKLIST.md
  Description:  The review checks — 6 core + 4 coverage-conditional + 3 cross-cutting — selected by manifest coverage state.
  Library ver:  0.2.0
  Last updated: 2026-07-03
─────────────────────────────── -->

# Review-Check Catalog — @webex/calling

> Start here → root [`AGENTS.md`](../AGENTS.md) (agent entry) · router [`src/ai-docs/SPEC_INDEX.md`](../src/ai-docs/SPEC_INDEX.md) · system [`ARCHITECTURE.md`](ARCHITECTURE.md). Then this doc at Review & Merge.
> Context-efficiency: link to canonical docs — don't duplicate them; load on demand, not upfront.

> Each finding records: severity (Blocking / Important / Medium / Minor), check id, file path, what's wrong,
> why it matters, a concrete fix. Any Blocking finding fails the gate.

## Core checks (always run)

| # | Check | What it verifies | Severity if it fails |
|---|---|---|---|
| C1 | Spec-currency + WHAT/WHY | Spec/docs changed in the same PR as code; every touched module spec is updated; every requirement (incl. ADDED) states WHAT and WHY; drift within threshold per `coverage-policy.defaults.yaml` | Blocking |
| C2 | Contract correctness | Provides/Requires delta in `CONTRACTS.md` and `.sdd/manifest.json` is real and complete; no undocumented breaking change to a public surface from `src/api.ts` | Blocking |
| C3 | Code-vs-spec match | Signatures, data-flow, and architecture claims in `*-spec.md` match the actual code (file path); public surface table in spec matches `src/api.ts` exports | Blocking |
| C4 | Test adequacy | Each acceptance criterion has a test with a positive AND a negative case; changed-line coverage ≥ 80% (branch) and ≥ 85% (statement/line); `yarn test:unit` passes | Important |
| C5 | Error handling + input validation | Untrusted input validated at public API boundaries; error classes used (not raw `Error`); errors emitted as typed events (call/line/client) or returned via `serviceErrorCodeHandler` (service modules); no silent swallowing | Important |
| C6 | Security baseline | No hardcoded secrets, tokens, keys; no PII in logs; auth via `SDKConnector` only; KMS/SCIM failure modes handled non-fatally; see `SECURITY.md` | Blocking |

## Coverage-conditional checks (run by the touched module's manifest coverage state)

| # | Check | When it applies | What it verifies | Severity |
|---|---|---|---|---|
| K1 | Regression guard | Modifying a DRAFT or PARTIAL module, or any MODIFIED/REMOVED requirement | A characterization baseline exists for the invariants the change claims NOT to alter; positive + negative cases both present | Blocking |
| K2 | Grounding | DRAFT or PARTIAL module touched | Claims cite real code (file path), not memory; uncovered public surfaces flagged `[NEEDS HUMAN INPUT]` | Important |
| K3 | Drift threshold | Any tracked module | Module drift is within its status threshold per `.sdd/coverage-policy.defaults.yaml` (DRAFT ≤ 25%, PARTIAL ≤ 15%, AUTHORITATIVE ≤ 5%) | Important |
| K4 | Coverage-state accuracy | Coverage-state change proposed in manifest | The recorded `coverageState` in `.sdd/manifest.json` matches the evidence; promotion from DRAFT → PARTIAL requires code-grounded validation; PARTIAL → AUTHORITATIVE requires full test evidence | Medium |

**Current module coverage states** (from `.sdd/manifest.json`):
- **DRAFT** — all 11 domain modules (10 specced + calling-package): spec was generated during migration, not yet independently validated.
- **PARTIAL** — all 5 infrastructure modules (SDKConnector, Logger, Events, Errors, common): read source directly, no spec yet.

When touching a DRAFT module: K1 + K2 + K3 apply.
When touching a PARTIAL module: K1 + K2 + K3 apply; K4 applies if you are promoting the state.

## Cross-cutting checks (apply at higher risk / autonomy)

| # | Check | What it verifies | Severity |
|---|---|---|---|
| X1 | Cross-model review | The artifact was validated by a different runtime than the one that generated it (generator ≠ validator). Required for spec promotions (DRAFT → PARTIAL or above) and for changes to registration/FSM/keepalive logic | Blocking when required |
| X2 | Observability | Logs use `Logger` module (never `console.*`); log format includes `{file, method}` context; no sensitive data logged; metrics submitted via `MetricManager` for relevant events | Medium |
| X3 | Rollout safety | No feature flags in this SDK — rollout is via semver; verify breaking change rule is honored; check `GETTING_STARTED.md` commands still accurate after dep changes | Important |

## How the set is selected

1. Always run the 6 core checks (C1–C6).
2. Add coverage-conditional checks (K1–K4) matching the touched modules' manifest `coverageState`.
3. Add cross-cutting checks (X1–X3) when the change is high-risk (registration, FSM, keepalive, public API surface, spec state promotion) or runs at higher autonomy.

## Calling-specific review notes

- **Registration/keepalive changes**: always require X1 (cross-model validation) — failures here cause devices to drop off Mobius silently.
- **XState FSM changes** (`callStateMachine.ts`, `mediaStateMachine.ts`): state name strings are not type-checked — verify all transition events against the updated FSM diagram in the spec.
- **`src/api.ts` changes**: any new export or signature change must update `CONTRACTS.md` and `SERVICE_STATE.md` in the same PR (C2).
- **Event enum changes** (`CALL_EVENT_KEYS`, `LINE_EVENTS`, `CALLING_CLIENT_EVENT_KEYS`): breaking for consumers; requires major version bump per `CONTRACTS.md` compatibility policy.
- **SCIM / KMS path changes**: verify non-fatal error handling is preserved (CID-R-004, CT-R-008 region).

## Output

- A compliance matrix (check → pass / warn / fail with file path) + severity-sorted findings + a verdict (Pass / Pass-with-warnings / Blocked).
- Draft only; a human posts findings to the PR.
