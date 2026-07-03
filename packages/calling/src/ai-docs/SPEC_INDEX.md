<!-- ───────────────────────────────
  Template:     Spec Index
  Template-ID:  spec-index
  Generates:    packages/calling/src/ai-docs/SPEC_INDEX.md
  Description:  Router — which docs to load for which task, module registry, intake routing.
  Library ver:  0.2.0
  Last updated: 2026-07-03
─────────────────────────────── -->

# Spec Index — @webex/calling

> Start here → root [`AGENTS.md`](../../AGENTS.md) (agent entry). This file is the router; system overview in [`calling-spec.md`](calling-spec.md). Load `AGENTS.md` + this file first; pull every other spec on demand.
> Context-efficiency: link to canonical specs — don't duplicate them; route to the minimum needed per task.

> AI agent entry point after `AGENTS.md`. Load this once at session start; pull module specs on demand.
> **Source of truth:** code and co-located specs (`.sdd/manifest.json` when present mirrors this for machines).

## Module Registry

| Module | Responsibility | Coverage state | Start here |
|---|---|---|---|
| `src/` | Package public API, factory functions, shared infrastructure wiring | Partial | `src/ai-docs/calling-spec.md` |
| `CallingClient/` | Line registration, call lifecycle, Mobius server discovery, network resilience | Specced | `CallingClient/ai-docs/calling-client-spec.md` |
| `CallingClient/line/` | Telephony line — register/deregister, call initiation, incoming call forwarding, event emission | Specced | `CallingClient/line/ai-docs/line-spec.md` |
| `CallingClient/registration/` | Device registration with Mobius, keepalive web worker, failover/failback | Specced | `CallingClient/registration/ai-docs/registration-spec.md` |
| `CallingClient/calling/` | Call and CallManager — XState call/ROAP state machines, call lifecycle, supplementary services | Specced | `CallingClient/calling/ai-docs/calling-sub-spec.md` |
| `CallingClient/calling/CallerId/` | Caller identity resolution from SIP headers + async SCIM enrichment | Specced | `CallingClient/calling/CallerId/ai-docs/caller-id-spec.md` |
| `CallHistory/` | Call history records — fetch, update missed calls, delete, real-time session events | Specced | `CallHistory/ai-docs/call-history-spec.md` |
| `CallSettings/` | Call waiting, DND, call forwarding, voicemail settings (WXC/UCM strategy pattern) | Specced | `CallSettings/ai-docs/call-settings-spec.md` |
| `Contacts/` | CRUD for contacts and contact groups with KMS encryption and SCIM resolution | Specced | `Contacts/ai-docs/contacts-spec.md` |
| `Voicemail/` | Voicemail list, content, transcripts, read/unread state, deletion (WXC/BWRKS/UCM) | Specced | `Voicemail/ai-docs/voicemail-spec.md` |
| `Metrics/` | Centralized telemetry singleton for all calling SDK events | Specced | `Metrics/ai-docs/metrics-spec.md` |
| `SDKConnector/` | Singleton bridge to Webex JS SDK (HTTP + Mercury WebSocket) | Partial | Source: `SDKConnector/index.ts`, `SDKConnector/types.ts` |
| `Logger/` | Leveled structured logging wrapper | Partial | Source: `Logger/index.ts`, `Logger/types.ts` |
| `Events/` | Typed `EventEmitter` base class and all event type maps | Partial | Source: `Events/types.ts`, `Events/impl/index.ts` |
| `Errors/` | Custom error hierarchy — `ExtendedError`, `CallError`, `LineError`, `CallingClientError` | Partial | Source: `Errors/catalog/`, `Errors/types.ts` |
| `common/` | Shared types, constants, utilities (backend detection, error handlers, SCIM, XSI) | Partial | Source: `common/types.ts`, `common/Utils.ts`, `common/constants.ts` |

## Task Routing

| If the task is… | Load |
|---|---|
| Understanding the package architecture | `src/ai-docs/calling-spec.md` sections: Design Overview, Data Flow |
| Working in `CallingClient/` (registration, calls, lines) | `CallingClient/ai-docs/calling-client-spec.md` |
| Working in `CallingClient/line/` | `CallingClient/line/ai-docs/line-spec.md` |
| Working in `CallingClient/registration/` | `CallingClient/registration/ai-docs/registration-spec.md` |
| Working in `CallingClient/calling/` (Call, CallManager) | `CallingClient/calling/ai-docs/calling-sub-spec.md` |
| Working in `CallerId/` | `CallingClient/calling/CallerId/ai-docs/caller-id-spec.md` |
| Working in `CallHistory/` | `CallHistory/ai-docs/call-history-spec.md` |
| Working in `CallSettings/` | `CallSettings/ai-docs/call-settings-spec.md` |
| Working in `Contacts/` | `Contacts/ai-docs/contacts-spec.md` |
| Working in `Voicemail/` | `Voicemail/ai-docs/voicemail-spec.md` |
| Working in `Metrics/` | `Metrics/ai-docs/metrics-spec.md` |
| A new feature spanning multiple modules | Load affected module specs + `src/ai-docs/calling-spec.md` Public Surface section |
| Backend detection logic | `src/ai-docs/calling-spec.md` — Calling Backend Detection section |
| Event key changes | `Events/types.ts` (authoritative) + affected module spec's Public Surface |
| Error handling changes | `Errors/` source + affected module spec's Error Handling section |
| A new module creation | `src/ai-docs/calling-spec.md` + `../../AGENTS.md` task-routing templates |

## Intake Routing

```
What kind of change?
├─ New feature in existing module  -> load that module's spec + root AGENTS.md for pre-questions
├─ New module                      -> load src/ai-docs/calling-spec.md + root AGENTS.md new-module template
├─ Bug / defect                    -> load affected module spec + root AGENTS.md bug-fix template
├─ Modify public API               -> load affected module spec Public Surface + src/ai-docs/calling-spec.md
└─ Doc/spec backfill only          -> reconcile target spec, update module spec, run conformance
```

The root `AGENTS.md` pre-questions confirm backend scope, API changes, events, errors, and metrics before any code generation.

## Spec Registry

| Doc | Location | Purpose |
|---|---|---|
| Package spec | `src/ai-docs/calling-spec.md` | Package-level overview, public API surface, backend detection, shared infrastructure |
| Patterns | `../../ai-docs/patterns/` | Repo conventions (TypeScript, testing, events, error-handling, architecture) |
| Rules | `../../ai-docs/RULES.md` | Enforceable coding do/don'ts |
| Module specs | `<module>/ai-docs/<module-name>-spec.md` | Per-module canonical spec (see Module Registry above) |

## Phase-Based Loading Protocol

| Phase | Load |
|---|---|
| Orient | `../../AGENTS.md` + this file |
| Understand module | that module's `<module-name>-spec.md` |
| Build | module spec + `../../ai-docs/patterns/` + `../../ai-docs/RULES.md` |
| Verify | independent validation against module spec Requirements table |
