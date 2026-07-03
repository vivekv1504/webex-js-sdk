# Rules — @webex/calling

> Start here → root [`AGENTS.md`](../../AGENTS.md) (agent entry) · repo-wide rules digest: [`../RULES.md`](../RULES.md). Individual rule files live here — one per enforceable constraint.

## Purpose

`RULES.md` is the digest — all rules summarized. Files in this directory are the detail pages for rules complex enough to warrant a full explanation (rationale, how-to-follow, enforcement path, code examples).

## Index

| File | Rule summary |
|---|---|
| [sdk-access-via-sdkconnector.md](sdk-access-via-sdkconnector.md) | Never access the Webex SDK directly in domain modules — always use `SDKConnector` |

## When to add a rule here

- The rule has a non-obvious rationale or was born from a real incident.
- A code example is needed to make it actionable.
- The enforcement path (lint rule, review check, or both) needs explanation.

## Format

Each file uses the SDLC rule template structure:
1. **Rule** — one clear statement
2. **Why** — the rationale (incident, constraint, invariant)
3. **How to follow** — code examples (correct vs. wrong)
4. **Enforced by** — lint rule / review check

## Maintenance

- Add a row to the index above when a new rule file is created.
- Cross-reference: `../RULES.md` carries the repo-wide summary; `../adr/` carries architectural decisions.
