<!-- ───────────────────────────────
  Template:     Rule
  Template-ID:  rule
  Generates:    ai-docs/rules/<name>.md
  Description:  One enforceable repo rule — the rule, its rationale, how to follow it, and how it's enforced.
  Library ver:  0.2.0
  Last updated: 2026-07-03
─────────────────────────────── -->

# Rule: Access the Webex SDK only through SDKConnector

> Start here → repo root [`AGENTS.md`](../../AGENTS.md) (agent entry, carries the critical rules) · router [`src/ai-docs/SPEC_INDEX.md`](../../src/ai-docs/SPEC_INDEX.md). This is an `ai-docs/rules/` file; the folder README is `../rules/README.md`; the repo-wide rules digest is `../RULES.md`.
> Context-efficiency: link to canonical docs — don't duplicate them; one rule per file; defer to the linter where it enforces.

## Rule

Never access the Webex JS SDK (`webex` instance, `webex.request()`, or `webex.internal.mercury`) directly from a domain module. All Webex SDK access MUST go through the `SDKConnector` singleton at `src/SDKConnector/index.ts`.

## Why

`SDKConnector` is the sole SDK gateway for `@webex/calling` (see ADR-0001). Accessing the SDK directly in domain modules scatters auth/transport coupling across 10+ files — a single SDK API change then ripples everywhere, Mercury listener registration becomes impossible to deduplicate or audit for teardown, and unit tests require a real `webex` instance instead of a simple mock.

## How to follow

```ts
// In a domain module — correct
import {SDKConnector} from '../SDKConnector';

// HTTP request:
const response = await SDKConnector.getWebex().request({uri: '...', method: 'GET'});

// Register a Mercury listener:
SDKConnector.registerListener(MOBIUS_EVENT_KEYS.CALL_INCOMING, handleIncoming);

// Unregister on teardown:
SDKConnector.unregisterListener(MOBIUS_EVENT_KEYS.CALL_INCOMING, handleIncoming);
```

```ts
// NEVER in a domain module — wrong
import webex from '@webex/webex-core';       // ❌ direct SDK import
webex.request({...});                         // ❌ direct HTTP call
webex.internal.mercury.on('event', handler); // ❌ direct Mercury registration
```

In tests: mock `SDKConnector` (e.g., `jest.spyOn(SDKConnector, 'getWebex').mockReturnValue(mockWebex)`) — never instantiate a real `webex` in a unit test.

## Enforced by

Review only — no automated ESLint `no-restricted-imports` rule currently blocks direct `webex` imports in domain modules. Reviewers enforce this as check **C6** (security baseline) and **C3** (code-vs-spec match). Consider adding a `no-restricted-imports` ESLint rule scoped to `src/` excluding `src/SDKConnector/` to make this automatic.
