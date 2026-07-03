<!-- ───────────────────────────────
  Template:     Glossary
  Template-ID:  glossary
  Generates:    ai-docs/GLOSSARY.md
  Description:  Ubiquitous language — domain term → definition → authoritative code location.
  Library ver:  0.2.0
  Last updated: 2026-07-03
─────────────────────────────── -->

# Glossary — @webex/calling

> Start here → root [`AGENTS.md`](../AGENTS.md) (agent entry) · router [`src/ai-docs/SPEC_INDEX.md`](../src/ai-docs/SPEC_INDEX.md) · system [`ARCHITECTURE.md`](ARCHITECTURE.md). Then this doc; related: `CONTRACTS.md`.
> Context-efficiency: link to canonical docs — don't duplicate them; load on demand, not upfront.

> Read this before naming anything. Use the canonical name exactly; never introduce a synonym. Find a term
> in code that isn't here? Add it rather than guessing its meaning.

## Domain Terms

| Term | Definition (one or two sentences) | Authoritative location (file/type) | Notes / synonyms to avoid |
|---|---|---|---|
| `CallingClient` | The root domain object created by `createClient()`. Owns line/device management, server discovery, and the media engine init. | `src/CallingClient/CallingClient.ts`, `types.ts#ICallingClient` | not "WebexCalling", not "SDK" |
| `Line` | Represents a registered device endpoint. Each line manages registration lifecycle and owns a `CallManager` for call creation. | `src/CallingClient/line/Line.ts`, `types.ts#ILine` | not "device", not "endpoint" |
| `Registration` | Handles the Mobius device registration protocol — POST /devices, keepalive heartbeats, and retry/fallback. | `src/CallingClient/registration/` | not "deviceRegistration" |
| `CallManager` | Singleton within a `Line` that creates `Call` objects and routes incoming Mobius WSS events to the correct `Call` by `callId`. | `src/CallingClient/calling/callManager.ts` | not "callController", not "callHandler" |
| `Call` | A single call instance. Drives two concurrent XState FSMs (call state + media ROAP). Exposes the public call API (`answer`, `hold`, `end`, etc.). | `src/CallingClient/calling/call.ts`, `types.ts#ICall` | not "session" |
| `CallerId` | Sub-module that resolves caller display identity for incoming calls via SIP header priority + async SCIM enrichment. | `src/CallingClient/calling/CallerId/CallerId.ts` | not "callerInfo" (that's the resolved data object) |
| `CallerInfo` | The resolved data object returned by `CallerId.fetchCallerDetails()` — contains `displayName`, `avatarURL`, raw SIP identity. | `src/CallingClient/calling/CallerId/types.ts#CallerInfo` | not "caller", not "callerId" |
| `Mobius` | Webex's SIP signaling server. Handles device registration (POST /devices) and call control (POST /calls). Reached via HTTP through `SDKConnector`. | `src/CallingClient/registration/`, `src/CallingClient/calling/constants.ts` | not "signaling server" alone |
| `Mercury` | Webex's persistent WebSocket service. Delivers real-time Mobius SIP events and Janus session events to the SDK. | `src/SDKConnector/index.ts` (mercury.on/off) | not "WebSocket", not "WS" |
| `SDKConnector` | Infrastructure singleton that bridges domain modules to the Webex JS SDK: wraps `webex.request()` for HTTP and registers/unregisters Mercury listeners. | `src/SDKConnector/index.ts` | not "webex", not "sdk" (lowercase) |
| `Eventing<T>` | Generic typed EventEmitter base class extended by all domain objects. Enforces compile-time event key and payload types. | `src/Events/impl/index.ts` | not "EventEmitter" (that's the Node.js base) |
| `MetricManager` | Singleton that submits telemetry events (call, registration, voicemail, BNR, connection) to the Calling Analytics API. | `src/Metrics/` | not "analytics", not "telemetry singleton" |
| `LineEmitter` | The `Eventing<LineEventTypes>` instance owned by `Line` that emits `LINE_EVENTS.*` to the application. | `src/CallingClient/line/types.ts#LineEventTypes` | not "lineEvents" |
| `CallingBackend` | Enum discriminating the backend flavor: `WXC` (Webex Calling), `BWRKS` (BroadWorks), `UCM` (Unified CM). | `src/common/types.ts#CallingBackend` | not "platform", not "backend type" |
| `ROAP` | Real-time Object Access Protocol — the SDP offer/answer exchange protocol modeled by the media state machine. | `src/CallingClient/calling/mediaStateMachine.ts` | not "SDP exchange" alone |
| `KMS` | Webex Key Management Service. Provides encryption keys for `Contacts` field encryption/decryption. | `src/Contacts/ContactsClient.ts` (encryptionKeyUrl) | not "key server", not "encryption service" |
| `SCIM` | System for Cross-domain Identity Management. Used to resolve Webex user identities in `CallerId` enrichment and `Contacts` cloud lookup. | `src/CallingClient/calling/CallerId/CallerId.ts`, `src/Contacts/` | not "user lookup", not "directory" |
| `correlationId` | UUID per `Call` instance used to correlate Mobius events and SDK metrics back to a specific call. | `src/CallingClient/calling/call.ts` | not "callId" (Mobius-assigned) vs "correlationId" (SDK-generated) |
| `lineId` | UUID per `Line` instance generated at SDK init, sent in Mobius registration. | `src/CallingClient/line/Line.ts` | not "deviceId" |
| `RegistrationStatus` | Enum tracking registration state: `ACTIVE`, `INACTIVE`, `REFRESH_IN_PROGRESS`, `UNREGISTERED`. | `src/CallingClient/registration/types.ts#RegistrationStatus` | not "status" alone |
| `keepalive` | Periodic `POST /devices/{id}/keepalive` heartbeat issued from a Web Worker to keep the Mobius registration alive. | `src/CallingClient/registration/worker.ts` | not "ping", not "heartbeat" |
| `Web Worker` | Browser background thread (`registration.worker.ts`) used exclusively for keepalive heartbeats to avoid main-thread blocking. | `src/CallingClient/registration/worker.ts` | not "background thread" alone |

## Abbreviations & Acronyms

| Abbreviation | Expansion | Meaning in this repo |
|---|---|---|
| WXC | Webex Calling | The cloud-native Webex Calling backend |
| BWRKS | BroadWorks | The BroadWorks (on-prem / hybrid) backend; treated as WXC path in voicemail |
| UCM | Unified Communications Manager (Cisco) | The Cisco UCM on-prem backend |
| SIP | Session Initiation Protocol | Underlying call signaling protocol used by Mobius |
| SDP | Session Description Protocol | Media capability negotiation format inside ROAP |
| ROAP | Real-time Object Access Protocol | SDP offer/answer exchange over WebSocket; drives `mediaStateMachine` |
| FSM | Finite State Machine | XState-based state machines (`callStateMachine`, `mediaStateMachine`) |
| SCIM | System for Cross-domain Identity Management | User directory query protocol |
| KMS | Key Management Service | Webex encryption key service for Contacts |
| DTMF | Dual-Tone Multi-Frequency | Keypad tones sent via `Call.sendDigit()` |
| BNR | Background Noise Removal | Audio feature tracked via `MetricManager.submitBNRMetric()` |
| DND | Do Not Disturb | Call settings toggle managed by `CallSettings` |
| XSI | Xtended Services Interface | BroadWorks REST API used by `CallSettings` WXC call-waiting path |
| ICE | Interactive Connectivity Establishment | WebRTC NAT traversal; warmup step in `CallingClient.init()` |
| PII | Personally Identifiable Information | Must never appear in logs; see `SECURITY.md` |

## Context-Specific Meanings

| Term | Context / module | Meaning here |
|---|---|---|
| `callId` | `CallManager` / Mobius events | The Mobius-assigned call identifier (string UUID) used for event routing |
| `callId` | `Call.correlationId` field | The SDK-generated UUID used for metrics and internal correlation |
| `backend` | `CallSettings`, `Voicemail` | `CallingBackend` enum value (`WXC` / `BWRKS` / `UCM`) used to select strategy |
| `session` | `VoicemailClient` | `sessionStorage` cache entry holding the WXC voicemail list |
| `session` | `CallHistory` | A Janus call session event (unrelated to browser sessionStorage) |

## Deprecated / Renamed Terms

| Old term | Current term | Why renamed | Still appears in |
|---|---|---|---|
| `AGENTS.md` (per-module) | `*-spec.md` (module spec) | SDLC migration: per-module orientation moved to canonical specs | `src/*/ai-docs/AGENTS.md` (archived in `ai-docs/_archive/pre-sdlc-migration/`) |
| `ARCHITECTURE.md` (per-module) | `*-spec.md` (design sections) | SDLC migration: architecture sections moved into module specs | `src/*/ai-docs/ARCHITECTURE.md` (archived) |

## Maintenance
- When a new domain concept is introduced (new entity, event, state), add it here in the same change.
- Cross-reference: public-surface terms → `CONTRACTS.md`.
