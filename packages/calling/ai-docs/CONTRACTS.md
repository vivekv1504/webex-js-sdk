<!-- ───────────────────────────────
  Template:     Contracts Catalog
  Template-ID:  contracts
  Generates:    ai-docs/CONTRACTS.md
  Description:  Standing as-built public-surface catalog (Provides/Requires) + compatibility policy.
  Library ver:  0.2.0
  Last updated: 2026-07-03
─────────────────────────────── -->

# Contracts Catalog — @webex/calling

> Start here → root [`AGENTS.md`](../AGENTS.md) (agent entry) · router [`src/ai-docs/SPEC_INDEX.md`](../src/ai-docs/SPEC_INDEX.md) · system [`ARCHITECTURE.md`](ARCHITECTURE.md). Then this root contract index; detailed contracts live in each module spec. Machine source `.sdd/manifest.json`.
> Context-efficiency: link to canonical docs — don't duplicate them; load on demand, not upfront.

> Read before adding any public-facing surface — check here first. Machine source of truth: `.sdd/manifest.json`.

## Provides — Exported API & Types

Exported from `src/api.ts` — the sole public entry point for `@webex/calling`.

### Factory Functions

| Contract ID | Owner module | Symbol | Signature | Stability / deprecation | Schema / detail link | Defined at |
|---|---|---|---|---|---|---|
| `calling.createClient` | `CallingClient` | `createClient` | `(webex: WebexSDK, config: CallingClientConfig): ICallingClient` | stable | `src/CallingClient/types.ts#ICallingClient` | `src/api.ts` |
| `calling.createCallHistoryClient` | `CallHistory` | `createCallHistoryClient` | `(webex: WebexSDK, logger: ILogger): ICallHistory` | stable | `src/CallHistory/types.ts#ICallHistory` | `src/api.ts` |
| `calling.createCallSettingsClient` | `CallSettings` | `createCallSettingsClient` | `(webex: WebexSDK, calling: CallingBackend, logger: ILogger): ICallSettings` | stable | `src/CallSettings/types.ts#ICallSettings` | `src/api.ts` |
| `calling.createContactsClient` | `Contacts` | `createContactsClient` | `(webex: WebexSDK, logger: ILogger): IContacts` | stable | `src/Contacts/types.ts#IContacts` | `src/api.ts` |
| `calling.createVoicemailClient` | `Voicemail` | `createVoicemailClient` | `(webex: WebexSDK, calling: CallingBackend, logger: ILogger): IVoicemail` | stable | `src/Voicemail/types.ts#IVoicemail` | `src/api.ts` |

### Exported Interfaces

| Contract ID | Owner module | Symbol | Stability | Schema / detail link | Defined at |
|---|---|---|---|---|---|
| `calling.ICallingClient` | `CallingClient` | `ICallingClient` | stable | `calling-client-spec.md` | `src/CallingClient/types.ts` |
| `calling.ILine` | `CallingClient/line` | `ILine` | stable | `line-spec.md` | `src/CallingClient/line/types.ts` |
| `calling.ICall` | `CallingClient/calling` | `ICall` | stable | `calling-sub-spec.md` | `src/CallingClient/calling/types.ts` |
| `calling.ICallHistory` | `CallHistory` | `ICallHistory` | stable | `call-history-spec.md` | `src/CallHistory/types.ts` |
| `calling.ICallSettings` | `CallSettings` | `ICallSettings` | stable | `call-settings-spec.md` | `src/CallSettings/types.ts` |
| `calling.IContacts` | `Contacts` | `IContacts` | stable | `contacts-spec.md` | `src/Contacts/types.ts` |
| `calling.IVoicemail` | `Voicemail` | `IVoicemail` | stable | `voicemail-spec.md` | `src/Voicemail/types.ts` |

### Exported Types

| Contract ID | Owner module | Symbol | Stability | Defined at |
|---|---|---|---|---|
| `calling.Contact` | `Contacts` | `Contact` | stable | `src/Contacts/types.ts` |
| `calling.ContactGroup` | `Contacts` | `ContactGroup` | stable | `src/Contacts/types.ts` |
| `calling.CallForwardSetting` | `CallSettings` | `CallForwardSetting` | stable | `src/CallSettings/types.ts` |
| `calling.CallForwardAlwaysSetting` | `CallSettings` | `CallForwardAlwaysSetting` | stable | `src/CallSettings/types.ts` |
| `calling.VoicemailSetting` | `CallSettings` | `VoicemailSetting` | stable | `src/CallSettings/types.ts` |
| `calling.VoicemailResponseEvent` | `Voicemail` | `VoicemailResponseEvent` | stable | `src/Voicemail/types.ts` |

### Events

| Contract ID | Owner module | Event key | Direction | Payload type | Delivery | Compatibility | Defined at |
|---|---|---|---|---|---|---|---|
| `calling.LINE_EVENTS.REGISTERED` | `Line` | `LINE_EVENTS.REGISTERED` | publish | `ILine` | once per registration | stable | `src/CallingClient/line/types.ts` |
| `calling.LINE_EVENTS.UNREGISTERED` | `Line` | `LINE_EVENTS.UNREGISTERED` | publish | _(none)_ | once per deregistration | stable | `src/CallingClient/line/types.ts` |
| `calling.LINE_EVENTS.INCOMING_CALL` | `Line` | `LINE_EVENTS.INCOMING_CALL` | publish | `ICall` | per incoming call | stable | `src/CallingClient/line/types.ts` |
| `calling.LINE_EVENTS.ERROR` | `Line` | `LINE_EVENTS.ERROR` | publish | `LineError` | per error | stable | `src/CallingClient/line/types.ts` |
| `calling.LINE_EVENTS.RECONNECTING` | `Line` | `LINE_EVENTS.RECONNECTING` | publish | _(none)_ | on Mercury reconnect | stable | `src/CallingClient/line/types.ts` |
| `calling.LINE_EVENTS.RECONNECTED` | `Line` | `LINE_EVENTS.RECONNECTED` | publish | _(none)_ | on registration restored | stable | `src/CallingClient/line/types.ts` |
| `calling.CALL_EVENT_KEYS.CONNECT` | `Call` | `CALL_EVENT_KEYS.CONNECT` | publish | `CallId` | once per call | stable | `src/Events/types.ts` |
| `calling.CALL_EVENT_KEYS.DISCONNECT` | `Call` | `CALL_EVENT_KEYS.DISCONNECT` | publish | `CallId` | once per call | stable | `src/Events/types.ts` |
| `calling.CALL_EVENT_KEYS.PROGRESS` | `Call` | `CALL_EVENT_KEYS.PROGRESS` | publish | `CallId` | ringing / early media | stable | `src/Events/types.ts` |
| `calling.CALL_EVENT_KEYS.HELD` | `Call` | `CALL_EVENT_KEYS.HELD` | publish | `CallId` | on hold | stable | `src/Events/types.ts` |
| `calling.CALL_EVENT_KEYS.RESUMED` | `Call` | `CALL_EVENT_KEYS.RESUMED` | publish | `CallId` | on resume | stable | `src/Events/types.ts` |
| `calling.CALL_EVENT_KEYS.CALLER_ID` | `Call` | `CALL_EVENT_KEYS.CALLER_ID` | publish | `CallerIdDisplay` | async, after incoming | stable | `src/Events/types.ts` |
| `calling.CALL_EVENT_KEYS.CALL_ERROR` | `Call` | `CALL_EVENT_KEYS.CALL_ERROR` | publish | `CallError` | per error | stable | `src/Events/types.ts` |
| `calling.CALLING_CLIENT_EVENT_KEYS.ERROR` | `CallingClient` | `CALLING_CLIENT_EVENT_KEYS.ERROR` | publish | `CallingClientError` | per error | stable | `src/Events/types.ts` |
| `calling.CALLING_CLIENT_EVENT_KEYS.USER_SESSION_INFO` | `CallingClient` | `CALLING_CLIENT_EVENT_KEYS.USER_SESSION_INFO` | publish | `CallSessionEvent` | per Janus session event | stable | `src/Events/types.ts` |

## Requires — what this repo depends on

| Dependency (service / package / datastore) | What is consumed | Availability assumption | Fallback on failure | Version floor |
|---|---|---|---|---|
| `@webex/internal-media-core` (peer dep) | `RoapMediaConnection` — SDP/ICE/WebRTC media engine | must be present at init | Media failure → `call:disconnect` event | ≥ 2.22.1 |
| Webex JS SDK (`webex` instance) | HTTP requests via `webex.request()`, Mercury WebSocket via `webex.internal.mercury` | must be authenticated at init | All calls fail without valid webex instance | per monorepo peer |
| Mobius REST API | Device registration (POST /devices), call control (POST /calls) | Webex cloud SLA; primary + backup URIs | Fallback to backup Mobius on primary failure | current Webex API |
| Mercury WebSocket (Webex) | Real-time SIP and Janus events | Webex cloud SLA | Reconnection handled by Webex SDK; SDK emits `RECONNECTING`/`RECONNECTED` events | current Webex API |
| Janus REST API | Call history (GET /history), real-time session events | Webex cloud SLA | No fallback — history unavailable until reconnection | current Webex API |
| SCIM API | User identity enrichment for CallerId and Contacts | best-effort | Non-fatal — returns un-enriched data on failure | current Webex API |
| KMS (Webex encryption) | Encryption key URL for CLOUD contact fields | Webex cloud SLA | KMS failure surfaces as ContactResponse error | current Webex API |
| XSI Actions API | BroadWorks call-waiting toggle (CallSettings WXC path) | Webex cloud SLA | None — settings update fails if XSI unavailable | current XSI API |

## Compatibility & Deprecation Policy

- **Breaking-change rule:** No breaking change to an exported interface or factory function signature without a semver major bump. Breaking changes require a deprecation comment in the same minor release and removal no sooner than the next major.
- **Deprecation:** Deprecated surfaces are marked with a `@deprecated` JSDoc tag in `src/api.ts` and noted in the module spec. Removal happens in the next major version.
- **Additive changes** (new optional parameters, new exported types, new events) are non-breaking and may ship in a minor release.
- Internal symbols not exported from `src/api.ts` carry no compatibility guarantee.

## Maintenance

- When a public surface is added/changed/removed: update this catalog, the owning module spec summary, and `.sdd/manifest.json` in the same change.
- For incompatible changes: include the consumer transition/deprecation plan and summarize it in the Compatibility / deprecation column.
- Cross-reference: domain terms → `GLOSSARY.md`.
