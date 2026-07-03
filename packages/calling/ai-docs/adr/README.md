# Architecture Decision Records — @webex/calling

> Start here → root [`AGENTS.md`](../../AGENTS.md) (agent entry) · router [`src/ai-docs/SPEC_INDEX.md`](../../src/ai-docs/SPEC_INDEX.md). ADRs record architectural decisions that shaped `@webex/calling` — context, decision, alternatives, and consequences.

## How ADRs work here

- **One decision per file.** File name: `NNNN-kebab-title.md` (e.g., `0001-sdkconnector-singleton-gateway.md`).
- **Sequential numbering.** Never reuse a number. Superseded ADRs stay on disk — update their `Status` to `Superseded by ADR-XXXX` and create the new ADR.
- **Status values:** `Accepted` | `Superseded by ADR-XXXX` | `Deprecated` | `Proposed`.
- **When to write one:** Any decision that is non-obvious and hard to reverse: choosing a pattern, accepting a constraint, rejecting a seemingly reasonable alternative. One-liner decisions don't need an ADR.
- **Template:** Use the SDLC `adr` template from `SDLC-Skills-main/templates/component-repo/standing-docs/` as a base.

## Index

| # | Title | Status | Date |
|---|---|---|---|
| [ADR-0001](0001-sdkconnector-singleton-gateway.md) | SDKConnector as singleton gateway to Webex SDK | Accepted | 2026-07-03 |

## Maintenance

- Add a row to this index in the same change that creates the ADR file.
- Cross-reference: `ARCHITECTURE.md` references the decisions that shape the system-level topology.
