<!-- ───────────────────────────────
  Template:     Getting Started
  Template-ID:  getting-started
  Generates:    ai-docs/GETTING_STARTED.md
  Description:  Clone/build/run loop, config/secrets, and multi-repo workspace layout.
  Library ver:  0.2.0
  Last updated: 2026-07-03
─────────────────────────────── -->

# Getting Started — @webex/calling

> Start here → root [`AGENTS.md`](../AGENTS.md) (agent entry) · router [`src/ai-docs/SPEC_INDEX.md`](../src/ai-docs/SPEC_INDEX.md) · system [`ARCHITECTURE.md`](ARCHITECTURE.md). Then this doc to get a build/test loop running.
> Context-efficiency: link to canonical docs — don't duplicate them; load on demand, not upfront.

## Prerequisites

- **Node.js** 18.x or 20.x (LTS)
- **Yarn** (classic, v1.x) — the monorepo uses Yarn workspaces
- **Git**
- A valid **Webex developer account** is required only for integration/E2E tests, not for building or running unit tests
- **Browser** environment — `@webex/calling` uses browser globals (`window`, `Worker`, `sessionStorage`, `RTCPeerConnection`). It is not Node.js-compatible at runtime.

## Clone & Install

```bash
# Clone the webex-js-sdk monorepo
git clone https://github.com/webex/webex-js-sdk.git
cd webex-js-sdk

# Install all workspace dependencies
yarn install
```

All `packages/calling` dependencies (including `@webex/internal-media-core`) are resolved by Yarn workspaces from the monorepo root.

## Build / Run / Test

All commands are run from `packages/calling/` or the monorepo root (prefixed with `yarn workspace @webex/calling`).

| Task | Command (from `packages/calling/`) |
|---|---|
| Build TypeScript | `yarn build` |
| Build source only | `yarn build:src` |
| Unit tests (Jest, runInBand) | `yarn test:unit` |
| Lint (ESLint) | `yarn test:style` |
| Auto-fix lint | `yarn fix:lint` |
| Auto-fix formatting | `yarn fix:prettier` |
| Generate TypeDoc docs | `yarn build:docs` |
| E2E tests (Playwright) | `yarn test:e2e` |

From monorepo root: `yarn workspace @webex/calling test:unit`

## First-Run Verification

After `yarn build`, confirm the output exists:

```bash
ls packages/calling/dist/index.js
# → dist/index.js (TypeScript compiled output)
```

After `yarn test:unit`, all Jest tests should pass with `≥ 85%` statement/function/line coverage and `≥ 80%` branch coverage (thresholds set in `jest.config.js`).

## Configuration & Secrets

- `@webex/calling` does not require a local `.env` file for building or running unit tests.
- **Integration / E2E tests** require a valid Webex access token and org credentials. Obtain test credentials from the Webex Developer Portal or your team's test account vault — never hardcode them (see `SECURITY.md`).
- Runtime: the host application passes an initialized `webex` instance to `createClient()`. No credentials are held by the calling SDK itself.

## Multi-Repo Workspace Layout

`@webex/calling` is part of the `webex-js-sdk` monorepo. The relevant workspace layout:

```
webex-js-sdk/
  packages/
    calling/              # @webex/calling — this package
    internal-media-core/  # @webex/internal-media-core — peer dep (WebRTC media engine)
    common/               # shared utilities
    jest-config-legacy/   # Jest config shared across packages
  docs/
    calling/              # TypeDoc output (yarn build:docs)
```

- Related repos: the host web application (e.g., Webex Web) consumes `@webex/calling` as an npm package.
- Cross-repo dependencies: see `ARCHITECTURE.md` (Cross-Repo Dependency Graph).

## Where to Go Next

- Agent entry: `../AGENTS.md` · System shape: `ARCHITECTURE.md` · Routing: `src/ai-docs/SPEC_INDEX.md`
- Conventions: `patterns/` + `ai-docs/rules/` (and `RULES.md`).
- Module specs: `src/ai-docs/SPEC_INDEX.md` — per-module requirements, flows, and pitfalls.
