<!-- ───────────────────────────────
  Template:     RULES
  Template-ID:  rules
  Generates:    ai-docs/RULES.md
  Description:  Enforceable do/don't beyond AGENTS — coverage, autonomy, naming, logging, errors, testing, security, drift, secrets.
  Library ver:  0.2.0
  Last updated: 2026-07-03
─────────────────────────────── -->

# Rules — @webex/calling

> Start here → root [`AGENTS.md`](../AGENTS.md) (agent entry, carries the critical rules) · router [`src/ai-docs/SPEC_INDEX.md`](../src/ai-docs/SPEC_INDEX.md) · system [`ARCHITECTURE.md`](ARCHITECTURE.md). Then this doc; pattern detail in `patterns/`.
> Context-efficiency: link to canonical docs — don't duplicate them; load on demand, not upfront.

> These rules are checkable. Every MUST rule records its source requirement/risk, verification path, severity, and owner. Name the tool where one enforces a rule; say "review only" plus why otherwise.

## Coverage Map (which docs/specs to trust)

| Module | Coverage state | What it means here |
|---|---|---|
| `src/CallingClient/` | Specced | `calling-client-spec.md` + sub-specs are authoritative; cross-check code only for very recent changes |
| `src/CallHistory/` | Specced | `call-history-spec.md` is authoritative |
| `src/CallSettings/` | Specced | `call-settings-spec.md` is authoritative |
| `src/Contacts/` | Specced | `contacts-spec.md` is authoritative |
| `src/Voicemail/` | Specced | `voicemail-spec.md` is authoritative |
| `src/Metrics/` | Specced | `metrics-spec.md` is authoritative |
| `src/CallingClient/line/` | Specced | `line-spec.md` is authoritative |
| `src/CallingClient/registration/` | Specced | `registration-spec.md` is authoritative |
| `src/CallingClient/calling/` | Specced | `calling-sub-spec.md` is authoritative |
| `src/CallingClient/calling/CallerId/` | Specced | `caller-id-spec.md` is authoritative |
| `src/SDKConnector/` | Partial | Read source (`SDKConnector/index.ts`, `types.ts`) — no spec yet |
| `src/Logger/` | Partial | Read source (`Logger/index.ts`, `types.ts`) — no spec yet |
| `src/Events/` | Partial | Read source (`Events/types.ts`, `Events/impl/index.ts`) — no spec yet |
| `src/Errors/` | Partial | Read source (`Errors/catalog/`, `Errors/types.ts`) — no spec yet |
| `src/common/` | Partial | Read source (`common/Utils.ts`, `common/types.ts`) — no spec yet |

Router: [`src/ai-docs/SPEC_INDEX.md`](../src/ai-docs/SPEC_INDEX.md)

## Autonomy & Ask-First

- **May proceed:** style fixes, test additions, copy/doc tweaks within a single module, adding a new constant or utility with no public-API impact.
- **Ask first / plan + confirm:** new public method or event (changes `src/api.ts`), new module, schema/type changes, multi-module refactors, changes to error hierarchy or event enums, changes to registration/keepalive/FSM logic.
- **Never without explicit human approval:** pushing to remote, creating/closing PRs, publishing npm, deleting branches, resetting/force-pushing, modifying CI/CD pipelines.

## Naming

All rules derived from `src/` conventions — not generic style guides. Verification: ESLint + TypeScript compiler enforce most; the rest are review-only.

| Element | Convention | Example | Enforced by |
|---|---|---|---|
| Classes | PascalCase | `CallingClient`, `CallManager` | TypeScript |
| Interfaces | `I` prefix + PascalCase | `ICall`, `ILine`, `ICallingClient` | Review |
| Type aliases | PascalCase, no prefix | `CallId`, `CorrelationId` | Review |
| Enum names + members (constants) | `SCREAMING_SNAKE_CASE` | `CALL_EVENT_KEYS.PROGRESS` | Review |
| Enum names + members (value enums) | `PascalCase` name + members | `CallDirection`, `RegistrationStatus` | Review |
| Methods / functions | camelCase | `getLines()`, `makeCall()` | TypeScript |
| Constants | `SCREAMING_SNAKE_CASE` | `NETWORK_FLAP_TIMEOUT` | Review |
| Source files (main class) | PascalCase `.ts` | `CallingClient.ts`, `CallHistory.ts` | Review |
| Source files (sub-module) | camelCase `.ts` | `call.ts`, `callManager.ts` | Review |
| Test files | `*.test.ts`, co-located | `CallingClient.test.ts` | Jest config |
| Type definition files | `types.ts` per module | `CallingClient/types.ts` | Review |
| Constants files | `constants.ts` per module | `CallingClient/constants.ts` | Review |

Detail: [`patterns/typescript-patterns.md`](patterns/typescript-patterns.md)

## Logging

Use the `Logger` module (`src/Logger/index.ts`) — never `console.log/warn/error`.

Log format: `CALLING_SDK: <UTC timestamp>: [LEVEL]: file:<file> - method:<method> - message:<msg>`

| Level | Method | When to use |
|---|---|---|
| 1 `error` | `log.error()` | Blocking failures; device registration failed; unhandled exception |
| 2 `warn` | `log.warn()` | Recoverable issues, fallbacks, non-blocking errors |
| 3 `log` | `log.log()` | General operational messages; method entry/exit for non-critical paths |
| 4 `info` | `log.info()` | Normal operations, state transitions, method entry/exit (most used) |
| 5 `trace` | `log.trace()` | Full stack traces, raw payloads, deep debugging |

Levels are cumulative — default is `error` (1). App sets level via `setLogger(level, module)`.

Always pass `{file: <FILE_CONST>, method: '<methodName>'}` context object. File constants are defined in `CallingClient/constants.ts` (`CALLING_CLIENT_FILE`, `CALL_FILE`, `LINE_FILE`, etc.).

**Never log:** tokens, credentials, SIP URIs containing user portions, or any PII. See `## Secrets Policy`.

## Error Handling

- MUST use the error class hierarchy (`ExtendedError` → `CallError` / `LineError` / `CallingClientError`)
- MUST use factory functions (`createCallError`, `createLineError`, `createClientError`) — never `new Error()`
- MUST use `ERROR_TYPE` and `ERROR_LAYER` enums for classification
- MUST emit errors as typed events: `CALL_EVENT_KEYS.CALL_ERROR`, `LINE_EVENTS.ERROR`, `CALLING_CLIENT_EVENT_KEYS.ERROR`
- MUST include `ErrorContext` (`{file, method}`) in every error — source: `Errors/types.ts`
- MUST use `handleCallErrors()` / `handleCallingClientErrors()` / `handleRegistrationErrors()` for HTTP error mapping (source: `common/Utils.ts`)
- MUST use `serviceErrorCodeHandler` for service modules (Voicemail, CallHistory, CallSettings, Contacts) — returns structured response objects, NOT events
- NEVER throw raw `Error` — always use typed error classes
- NEVER swallow errors silently — emit, log, or propagate

Two distinct paths:
1. **Call/Line/Client errors** → typed error object → `emit(event, error)` → application listener
2. **Service module errors** → `serviceErrorCodeHandler` → structured response returned directly to caller (no event emitted)

Detail: [`patterns/error-handling-patterns.md`](patterns/error-handling-patterns.md)

## Imports / Dependencies

Import ordering (3-tier):
1. External packages (`xstate`, `async-mutex`, `uuid`)
2. Internal `@webex/*` packages (`@webex/internal-media-core`)
3. Relative imports: parent → sibling → child

Public API exports: through `src/api.ts` only — do not re-export internal types from `api.ts`.
Types: export from module's `types.ts`; use `export type` for type-only exports.
New dependencies: requires lead approval before adding to `package.json`.

NEVER: cross-layer imports that violate the `App → Module → SDKConnector → Webex SDK` layering. Call/line classes must not import `webex` directly — always go through `SDKConnector`.

## Testing

- MUST use Jest + jsdom environment
- MUST co-locate test files as `ModuleName.test.ts` alongside source
- MUST use `getTestUtilsWebex()` from `src/common/testUtil.ts` for mock Webex instances
- MUST use `jest.fn()` / `jest.spyOn()` for mocking (not Sinon — even though it's a dev dep)
- MUST mock `@webex/internal-media-core` at the top of call-related test files
- MUST achieve ≥ 85% lines/functions/statements coverage and ≥ 80% branch coverage (global)
- MUST clean up with `jest.clearAllMocks()` in `beforeEach`/`afterEach`
- MUST cover both success and failure paths for every public method
- Test behavior and event emissions — NOT implementation details or private methods
- NEVER leave unmocked external dependencies in unit tests

Commands: `yarn test:unit` · `yarn test:style` · `yarn build` — all must pass before merge.

Detail: [`patterns/testing-patterns.md`](patterns/testing-patterns.md)

## Security

- No hardcoded API keys, tokens, secrets, passwords, or certificates — ever
- Never log tokens, credentials, or PII (see `## Secrets Policy`)
- Use `SDKConnector.request()` for all API calls — never call `webex.request()` directly from domain modules
- Authorization is managed by the Webex SDK; the calling SDK must not bypass or replicate auth flows
- All user-facing inputs (destination addresses, contact data) must be validated at the public API boundary before sending to Mobius or service APIs

## Spec-Currency & Drift Thresholds

- Update the spec/docs in the SAME PR as the code change (spec-currency rule)
- **Specced modules:** ≤ 5% drift — any code change in a specced module MUST update the corresponding `*-spec.md`
- **Partial-coverage modules:** ≤ 25% drift — best-effort; no spec update required but note the gap in the PR description
- When a spec section conflicts with the code, trust the code — then fix the spec

## Secrets Policy

No hardcoded secrets, tokens, keys, or connection strings — ever. All secrets come from the Webex SDK auth layer at runtime. CI runs secret scanning; a detected secret fails the build. Never log sensitive data (see `## Logging`).

## Concurrency & Async

- MUST use `async-mutex` (`async-mutex` pkg) for critical sections: registration calls, line creation in `CallingClient`
- MUST use `async/await` over raw `.then()` chains
- Web Worker keepalive (`registration.worker.ts`) runs off the main thread — do not block it with synchronous operations
- XState FSM transitions are synchronous — all side effects (Mobius API calls) happen in action handlers, not transition guards
- Mercury WebSocket callbacks are single-threaded — event processing in `CallManager.dequeueWsEvents()` is sequential
- Event listeners MUST be unregistered (`off()` / `SDKConnector.unregisterListener()`) on teardown to prevent memory leaks

Detail: [`patterns/architecture-patterns.md`](patterns/architecture-patterns.md) (Concurrency Control section)

## Maintenance

- Add a rule when a review correction recurs; remove it when a lint rule starts enforcing it.
- Cross-reference: patterns → `patterns/`; spec details → `src/ai-docs/SPEC_INDEX.md`.
- Pattern docs: [`architecture-patterns.md`](patterns/architecture-patterns.md) · [`error-handling-patterns.md`](patterns/error-handling-patterns.md) · [`event-patterns.md`](patterns/event-patterns.md) · [`testing-patterns.md`](patterns/testing-patterns.md) · [`typescript-patterns.md`](patterns/typescript-patterns.md)
