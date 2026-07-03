# Archive: pre-SDLC-migration AI Docs

> This directory contains the original `AGENTS.md` and `ARCHITECTURE.md` files from each `@webex/calling` module's `ai-docs/` folder, preserved as-is at the time of the SDLC-Templates component-repo migration (2026-07-03).

## Why preserved

These files were the source material for the SDLC migration. They are retained here for:
- Auditing what content was migrated vs. what is new
- Rollback reference if a migrated spec is found to be inaccurate
- Historical context for AI agents reviewing git blame

## Original file locations → archive names

| Archive file | Original location |
|---|---|
| `src_AGENTS.md` | `src/ai-docs/AGENTS.md` |
| `src_ARCHITECTURE.md` | `src/ai-docs/ARCHITECTURE.md` |
| `CallingClient_AGENTS.md` | `src/CallingClient/ai-docs/AGENTS.md` |
| `CallingClient_ARCHITECTURE.md` | `src/CallingClient/ai-docs/ARCHITECTURE.md` |
| `calling_AGENTS.md` | `src/CallingClient/calling/ai-docs/AGENTS.md` |
| `calling_ARCHITECTURE.md` | `src/CallingClient/calling/ai-docs/ARCHITECTURE.md` |
| `CallerId_AGENTS.md` | `src/CallingClient/calling/CallerId/ai-docs/AGENTS.md` |
| `line_AGENTS.md` | `src/CallingClient/line/ai-docs/AGENTS.md` |
| `line_ARCHITECTURE.md` | `src/CallingClient/line/ai-docs/ARCHITECTURE.md` |
| `registration_AGENTS.md` | `src/CallingClient/registration/ai-docs/AGENTS.md` |
| `registration_ARCHITECTURE.md` | `src/CallingClient/registration/ai-docs/ARCHITECTURE.md` |
| `CallHistory_AGENTS.md` | `src/CallHistory/ai-docs/AGENTS.md` |
| `CallHistory_ARCHITECTURE.md` | `src/CallHistory/ai-docs/ARCHITECTURE.md` |
| `CallSettings_AGENTS.md` | `src/CallSettings/ai-docs/AGENTS.md` |
| `CallSettings_ARCHITECTURE.md` | `src/CallSettings/ai-docs/ARCHITECTURE.md` |
| `Contacts_AGENTS.md` | `src/Contacts/ai-docs/AGENTS.md` |
| `Contacts_ARCHITECTURE.md` | `src/Contacts/ai-docs/ARCHITECTURE.md` |
| `Voicemail_AGENTS.md` | `src/Voicemail/ai-docs/AGENTS.md` |
| `Voicemail_ARCHITECTURE.md` | `src/Voicemail/ai-docs/ARCHITECTURE.md` |
| `Metrics_AGENTS.md` | `src/Metrics/ai-docs/AGENTS.md` |
| `Metrics_ARCHITECTURE.md` | `src/Metrics/ai-docs/ARCHITECTURE.md` |

## What replaced them

Each original AGENTS.md + ARCHITECTURE.md pair was migrated into a canonical `*-spec.md` in the SDLC module-spec format. See `src/ai-docs/SPEC_INDEX.md` for the current module registry and spec locations.

The originals are NOT deleted — they remain in their source locations alongside the new `*-spec.md` files. This archive is a copy.
