<!-- ───────────────────────────────
  Template:     ADR
  Template-ID:  adr
  Generates:    ai-docs/adr/NNNN-<kebab-title>.md
  Description:  Standing architecture decision record — context, decision, alternatives rejected, consequences.
  Library ver:  0.2.0
  Last updated: 2026-07-03
─────────────────────────────── -->

# ADR-0001 — SDKConnector as the sole gateway to the Webex SDK

> Start here → repo root [`AGENTS.md`](../../AGENTS.md) (agent entry) · router [`src/ai-docs/SPEC_INDEX.md`](../../src/ai-docs/SPEC_INDEX.md) · system [`ARCHITECTURE.md`](../ARCHITECTURE.md). This is a standing `ai-docs/adr/` decision record; the folder README explains numbering/supersession.
> Context-efficiency: link to canonical docs — don't duplicate them; one decision per file.

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-07-03 |
| Deciders | @webex/calling maintainers |
| Supersedes / Superseded by | none |
| Generated from | `adr` @ SDLC template library `0.2.0` |

## Context

`@webex/calling` is a TypeScript library that runs inside a host web application alongside the Webex JS SDK. It makes HTTP requests to Webex APIs (Mobius, Janus, SCIM, KMS, XSI) and subscribes to real-time events from the Mercury WebSocket. The Webex JS SDK provides both the HTTP transport (`webex.request()`) and the Mercury client (`webex.internal.mercury`).

Without a stated access pattern, every domain module (`CallingClient`, `CallHistory`, `CallSettings`, `Contacts`, `Voicemail`, `Registration`, `Call`, `CallerId`) would import `webex` directly. This would scatter Webex SDK coupling across 10+ files, make the auth/transport layer impossible to swap or mock in isolation, and create duplicate Mercury listener registration logic in every module.

The actual code already routes all Webex SDK access through a single `SDKConnector` class (`src/SDKConnector/index.ts`) — this ADR records that arrangement as a deliberate constraint.

## Decision

All access to the Webex JS SDK from within `@webex/calling` MUST go through the `SDKConnector` singleton:

```
Domain modules → SDKConnector.getWebex().request() → Webex SDK HTTP
Domain modules → SDKConnector.registerListener()    → Mercury WebSocket
```

- `SDKConnector` is initialized once at `CallingClient` creation with the host-provided `webex` instance.
- No domain module may hold a direct reference to the `webex` instance.
- No domain module may call `webex.request()` or `webex.internal.mercury.on()` directly — always via `SDKConnector`.
- In tests: mock `SDKConnector` at the boundary — domain module tests never need a real `webex` instance.

## Alternatives Considered

| Alternative | Pros | Cons | Why rejected |
|---|---|---|---|
| Pass `webex` as a constructor parameter to every domain module | Simple, explicit dependency | Webex SDK coupling scattered across every module; auth/transport changes ripple everywhere; Mercury listener deduplication impossible | Rejected — defeats the single-boundary goal and makes mocking complex |
| Use a dependency injection container | Testable, loosely coupled | Adds a framework dependency; overkill for a focused SDK with 5 domain modules | Rejected — singleton pattern is sufficient and already in place |
| Each module registers its own Mercury listeners directly | Module-local ownership | Multiple `mercury.on()` calls for the same event; risk of duplicate processing; teardown order bugs | Rejected — `SDKConnector.registerListener()` provides a single registration point with a deduplication guarantee |

## Consequences

- **Positive:** Webex SDK coupling is isolated to one file (`SDKConnector/index.ts`); auth token management, request retry, and Mercury multiplexing are centralized; domain module unit tests are simple (mock `SDKConnector`); Mercury listener teardown is explicit and auditable.
- **Negative / cost:** Any new Webex SDK surface (new API, new Mercury event type) must be exposed through `SDKConnector` before domain modules can use it — adds one indirection layer.
- **Agents must:** Never import `webex` directly in a domain module; always call `SDKConnector.getWebex()` or `SDKConnector.registerListener()`; route new Mercury event subscriptions through `SDKConnector.registerListener()`.

## Revisit When

- The `SDKConnector` interface grows so large it no longer has a single, clear responsibility (e.g., more than HTTP + Mercury).
- A second, independent Webex SDK instance is needed in the same calling session (highly unlikely given the single-device-per-session model).
