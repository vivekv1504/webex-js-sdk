<!-- ───────────────────────────────
  Template:     Security Baseline
  Template-ID:  security
  Generates:    ai-docs/SECURITY.md
  Description:  Standing security posture — trust boundaries, authn/authz, secret handling, data classification.
  Library ver:  0.2.0
  Last updated: 2026-07-03
─────────────────────────────── -->

# Security Baseline — @webex/calling

> Start here → root [`AGENTS.md`](../AGENTS.md) (agent entry) · router [`src/ai-docs/SPEC_INDEX.md`](../src/ai-docs/SPEC_INDEX.md) · system [`ARCHITECTURE.md`](ARCHITECTURE.md). Then this doc; per-feature security design lives in each module spec.
> Context-efficiency: link to canonical docs — don't duplicate them; load on demand, not upfront.

> Read before changing anything that touches input, identity, data, or external calls. Don't weaken a
> documented control without an explicit, approved decision (record it as an ADR in `ai-docs/adr/`).

## Trust Boundaries

| Boundary | Untrusted side | Trusted side | What is enforced at the crossing |
|---|---|---|---|
| Public API surface (`src/api.ts`) | Host application / user input | `@webex/calling` SDK | Input validation (destination address format, required parameters like `contactId` for CLOUD contacts, SCIM batch size ≤ 50); see each module spec for boundary-specific rules |
| SDKConnector → Webex SDK | `@webex/calling` domain modules | Webex JS SDK + Mobius | Auth delegated entirely to `webex.request()` — calling SDK never constructs or inspects tokens |
| Mercury WebSocket events | Mobius / Janus server-pushed events | SDK internal handlers | Event schema is consumed as-is; no additional signature verification in calling SDK (Webex SDK handles connection-level auth) |
| SCIM API responses | SCIM service | `CallerId`, `Contacts` | SCIM failure is treated as non-fatal; no PII from SCIM is logged |
| KMS responses | Webex KMS | `Contacts` | KMS decryption failure surfaces as an error response; decrypted contact fields must never be logged |

## Authentication & Authorization Model

- **Authentication:** Entirely delegated to the Webex JS SDK layer. `SDKConnector` wraps `webex.request()` (`src/SDKConnector/index.ts`) — all HTTP requests carry auth tokens managed by the Webex SDK. The calling SDK never reads, stores, or logs access tokens.
- **Authorization:** Webex SDK enforces auth on all outbound HTTP calls (Mobius, Janus, SCIM, KMS, XSI). The calling SDK has no RBAC layer of its own — it relies on the Webex token scopes set by the host application.
- **Default posture:** If `SDKConnector` is not initialized (no valid `webex` instance), all requests fail immediately — there is no anonymous-access path.

## Secret & Credential Handling

- Secrets source: Webex SDK auth layer at runtime — obtained from the Webex OAuth flow by the host app; never hardcoded in `@webex/calling`.
- Injection: the host app passes an initialized `webex` instance to `createClient()` / factory functions; credentials are never passed as parameters to calling SDK methods.
- Rotation: managed by Webex SDK token refresh logic — calling SDK is unaffected by token rotation.
- **Hard rule:** Never commit secrets, tokens, keys, or connection strings. Never log them. CI secret scanning enforces this at build time.

## Data Classification & Handling

| Data class | Examples | Storage rule | Logging rule | In transit |
|---|---|---|---|---|
| Auth tokens / credentials | Webex access tokens, refresh tokens | Never stored in calling SDK (owned by Webex SDK) | Never log — not even partial values | TLS via `webex.request()` |
| PII — user identity | Display names, email addresses, avatar URLs from SCIM | In-memory only during call lifetime; not persisted | Never log raw PII fields | TLS via SCIM API call |
| SIP URIs | `sip:user@domain` from call headers | In-memory during call resolution; not persisted | Log domain only if needed — never log user portion | TLS via Mobius |
| Contact data (CLOUD) | Contact fields encrypted with KMS | Encrypted at rest in Webex cloud (KMS-managed); in-memory after decryption during session | Never log decrypted contact field values | TLS; KMS encryption end-to-end |
| Voicemail content URL | Signed URL returned by Webex | In-memory only; session-scoped cache (`sessionStorage`) — no sensitive payload in URL logged | Log URL presence (boolean), not the URL value | TLS |
| `correlationId` / `lineId` | SDK-generated UUIDs | In-memory per session | Safe to log — no PII | TLS |
| Call metrics | Registration/call durations, error codes, Mobius URL | Submitted to Calling Analytics API; no PII | Log metric type/outcome, not user-linked fields | TLS via `webex.request()` |

## Input Validation & Output Encoding Posture

- Validate at the public API boundary (`src/api.ts` factory calls and public methods):
  - Destination address format before passing to Mobius.
  - `contactId` presence required for CLOUD contact type (see `contacts-spec.md` CT-R-002).
  - SCIM batch size ≤ 50 enforced before request (CT-R-003).
- Internal calls between modules assume validated inputs — do not re-validate at every layer.
- No HTML/DOM rendering in `@webex/calling` — output encoding is not applicable.

## Transport & Headers

- HTTPS/TLS everywhere — all HTTP requests go through `webex.request()` which enforces TLS.
- No direct `fetch()` calls from domain modules — always through `SDKConnector`.
- Mercury WebSocket uses WSS (enforced by Webex SDK connection layer).
- No CORS/CSRF posture owned by the calling SDK — it does not serve HTTP endpoints.

## Known Sensitive Areas & Accepted Risks

| Area | Risk | Mitigation / why accepted | Owner |
|---|---|---|---|
| `sessionStorage` voicemail cache (WXC) | Browser-accessible; cleared on session end but shared within the browser tab | Only stores voicemail list metadata (IDs, timestamps, flags) — no audio content or decrypted PII; aligned with Webex web app conventions | calling SDK maintainers |
| SCIM enrichment in CallerId | SCIM may return display names / photos for callers | Data stays in-memory for call lifetime; never persisted or logged; used only for UI display | calling SDK maintainers |
| Web Worker keepalive | Worker runs in background and makes periodic HTTP requests | Only sends `POST /devices/{id}/keepalive` with no body; auth handled by Webex SDK; worker is terminated on `line.deregister()` | calling SDK maintainers |

## Reporting & Review

- Security-relevant changes (touching auth flows, token handling, PII, input validation, transport, or KMS/encryption) require review from the calling SDK maintainers and the Webex security team.
- Suspected vulnerabilities: report via the Webex Bug Bounty program or internally via the Cisco PSIRT process.
- Cross-reference: per-module security design lives in each `*-spec.md` (Security / Error Handling sections).
