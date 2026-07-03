<!-- ───────────────────────────────
  Template:     Service State (living)
  Template-ID:  service-state
  Generates:    ai-docs/SERVICE_STATE.md
  Description:  Living as-built registry — current endpoints/events/stores/deps/limits/metrics/flags; read first to avoid duplicates.
  Library ver:  0.2.0
  Last updated: 2026-07-03
─────────────────────────────── -->

# Service State (living) — @webex/calling

> Start here → root [`AGENTS.md`](../AGENTS.md) (agent entry) · router [`src/ai-docs/SPEC_INDEX.md`](../src/ai-docs/SPEC_INDEX.md) · system [`ARCHITECTURE.md`](ARCHITECTURE.md). Read this FIRST before adding a surface; stable contracts in `CONTRACTS.md`.
> Context-efficiency: link to canonical docs — don't duplicate them; load on demand, not upfront.

> Source of truth for "does X already exist?" Keep current in the same change that adds/removes a surface.

## Current Exported Factory Functions

| Symbol | Signature (abbreviated) | Defined at |
|---|---|---|
| `createClient` | `(webex, config) → ICallingClient` | `src/api.ts`, `src/CallingClient/CallingClient.ts` |
| `createCallHistoryClient` | `(webex, logger) → ICallHistory` | `src/api.ts`, `src/CallHistory/CallHistory.ts` |
| `createCallSettingsClient` | `(webex, calling, logger) → ICallSettings` | `src/api.ts`, `src/CallSettings/CallSettings.ts` |
| `createContactsClient` | `(webex, logger) → IContacts` | `src/api.ts`, `src/Contacts/ContactsClient.ts` |
| `createVoicemailClient` | `(webex, calling, logger) → IVoicemail` | `src/api.ts`, `src/Voicemail/Voicemail.ts` |

## Current Exported Interfaces

| Symbol | Defined at | Detail spec |
|---|---|---|
| `ICallingClient` | `src/CallingClient/types.ts` | `calling-client-spec.md` |
| `ILine` | `src/CallingClient/line/types.ts` | `line-spec.md` |
| `ICall` | `src/CallingClient/calling/types.ts` | `calling-sub-spec.md` |
| `ICallHistory` | `src/CallHistory/types.ts` | `call-history-spec.md` |
| `ICallSettings` | `src/CallSettings/types.ts` | `call-settings-spec.md` |
| `IContacts` | `src/Contacts/types.ts` | `contacts-spec.md` |
| `IVoicemail` | `src/Voicemail/types.ts` | `voicemail-spec.md` |

## Current Exported Types

| Symbol | Defined at |
|---|---|
| `Contact`, `ContactGroup` | `src/Contacts/types.ts` |
| `CallForwardSetting`, `CallForwardAlwaysSetting`, `VoicemailSetting` | `src/CallSettings/types.ts` |
| `VoicemailResponseEvent` | `src/Voicemail/types.ts` |

## Current Events (published by SDK)

### Line Events (`LINE_EVENTS`)

| Event key | Owner | Payload | Trigger | Defined at |
|---|---|---|---|---|
| `LINE_EVENTS.REGISTERED` | `Line` | `ILine` | Registration succeeded | `src/CallingClient/line/types.ts` |
| `LINE_EVENTS.UNREGISTERED` | `Line` | _(none)_ | Line deregistered | `src/CallingClient/line/types.ts` |
| `LINE_EVENTS.INCOMING_CALL` | `Line` | `ICall` | Incoming Mobius INVITE received | `src/CallingClient/line/types.ts` |
| `LINE_EVENTS.ERROR` | `Line` | `LineError` | Registration or line error | `src/CallingClient/line/types.ts` |
| `LINE_EVENTS.CONNECTING` | `Line` | _(none)_ | Re-registration in progress | `src/CallingClient/line/types.ts` |
| `LINE_EVENTS.RECONNECTING` | `Line` | _(none)_ | Mercury WebSocket reconnect attempt | `src/CallingClient/line/types.ts` |
| `LINE_EVENTS.RECONNECTED` | `Line` | _(none)_ | Registration restored after reconnect | `src/CallingClient/line/types.ts` |

### Call Events (`CALL_EVENT_KEYS`)

| Event key | Owner | Payload | Trigger | Defined at |
|---|---|---|---|---|
| `CALL_EVENT_KEYS.CONNECT` | `Call` | `CallId` | Call established | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.DISCONNECT` | `Call` | `CallId` | Call ended | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.PROGRESS` | `Call` | `CallId` | Call ringing / early media | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.ESTABLISHED` | `Call` | `CallId` | Media established | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.HELD` | `Call` | `CallId` | Call held | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.RESUMED` | `Call` | `CallId` | Call resumed from hold | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.CALLER_ID` | `Call` | `CallerIdDisplay` | CallerID async enrichment resolved | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.REMOTE_MEDIA` | `Call` | `MediaStreamTrack` | Remote media track available | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.CALL_ERROR` | `Call` | `CallError` | Call-level error | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.HOLD_ERROR` | `Call` | `CallError` | Hold operation failed | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.RESUME_ERROR` | `Call` | `CallError` | Resume operation failed | `src/Events/types.ts` |
| `CALL_EVENT_KEYS.TRANSFER_ERROR` | `Call` | `CallError` | Transfer operation failed | `src/Events/types.ts` |

### CallingClient Events (`CALLING_CLIENT_EVENT_KEYS`)

| Event key | Owner | Payload | Trigger | Defined at |
|---|---|---|---|---|
| `CALLING_CLIENT_EVENT_KEYS.ERROR` | `CallingClient` | `CallingClientError` | Client-level error | `src/Events/types.ts` |
| `CALLING_CLIENT_EVENT_KEYS.USER_SESSION_INFO` | `CallingClient` | `CallSessionEvent` | Janus session event received | `src/Events/types.ts` |
| `CALLING_CLIENT_EVENT_KEYS.OUTGOING_CALL` | `CallingClient` | `string` (callId) | Outbound call created | `src/Events/types.ts` |
| `CALLING_CLIENT_EVENT_KEYS.ALL_CALLS_CLEARED` | `CallingClient` | _(none)_ | All active calls cleared | `src/Events/types.ts` |
| `CALLING_CLIENT_EVENT_KEYS.MOBIUS_SOCKET_CONNECTED` | `CallingClient` | _(none)_ | Mobius WebSocket connected | `src/Events/types.ts` |
| `CALLING_CLIENT_EVENT_KEYS.MOBIUS_SOCKET_DISCONNECTED` | `CallingClient` | _(varies)_ | Mobius WebSocket disconnected | `src/Events/types.ts` |

## Data Stores

| Store | Purpose | Owned by this SDK? |
|---|---|---|
| `sessionStorage` (browser) | WXC voicemail list cache per session | No — browser-owned; SDK writes/reads entries |
| In-memory: `CallingClient.lineDict` | Active `ILine` objects by `lineId` | Yes — per-session in-memory only |
| In-memory: `CallManager.callCollection` | Active `ICall` objects by `callId` | Yes — per-session in-memory only |
| In-memory: `Registration.primaryServers` / `backupServers` | Mobius server URL lists | Yes — per-session; refreshed on re-registration |
| In-memory: `ContactsClient.encryptionKeyUrl` | KMS key URL for CLOUD contacts | Yes — per-session; set once, never re-fetched |

## External Dependencies

| Dependency | Used for | Timeout / retry | Circuit breaker / fallback |
|---|---|---|---|
| Mobius REST API | Registration (POST /devices), call control (POST /calls) | Per `webex.request()` defaults; 3 retries on 429 | Backup Mobius server list (fallback after primary failures) |
| Mercury WebSocket | Real-time SIP and Janus events | Managed by Webex SDK reconnect logic | SDK emits RECONNECTING/RECONNECTED; SDK retries registration |
| Janus REST API | Call history, real-time session events | Per `webex.request()` defaults | No local fallback — history unavailable until connection restored |
| SCIM API | CallerID enrichment, CLOUD contact lookup | Per `webex.request()` defaults | Non-fatal fallback — returns un-enriched data |
| KMS | CLOUD contact field encryption/decryption | Per Webex SDK KMS handling | Error surfaced as ContactResponse error |
| XSI Actions API | CallSettings WXC call-waiting | Per `webex.request()` defaults | Settings update fails; no local fallback |
| `@webex/internal-media-core` | WebRTC SDP/ICE media engine | N/A (in-process) | Media failure → `call:disconnect` event |

## Rate Limits & Quotas

| Surface | Limit | Scope |
|---|---|---|
| SCIM contact batch (CT-R-003) | ≤ 50 contacts per batch request | Per request |
| Mobius keepalive interval | Set by Mobius `keepaliveInterval` response | Per registered device |
| Mobius 429 retry | Three distinct retry paths: immediate retry, rescheduled retry, backup server | Per registration attempt |

## Key Metrics & Performance Targets

| Signal | Target | Where measured |
|---|---|---|
| Registration success rate | Webex SLA | `MetricManager.submitRegistrationMetric()` → Calling Analytics API |
| Call setup success rate | Webex SLA | `MetricManager.submitCallMetric()` → Calling Analytics API |
| Voicemail operation success rate | Webex SLA | `MetricManager.submitVoicemailMetric()` → Calling Analytics API |
| BNR toggle success | N/A | `MetricManager.submitBNRMetric()` → Calling Analytics API |
| Keepalive timer | ≤ 1% missed heartbeats | Implicit in registration health |

## Feature Flags (current)

No runtime feature flags. Capability selection is done at init time via the `CallingBackend` enum (`WXC` / `BWRKS` / `UCM`) passed to factory functions.

## Maintenance

- Update the relevant row in the same change that adds/changes/removes a surface, dependency, limit, or flag.
- Cross-reference: stable contracts → `CONTRACTS.md`; security posture → `SECURITY.md`; domain terms → `GLOSSARY.md`.
