# Implementation Plan: GitHub Copilot Provider Replacement

**Branch**: `001-copilot-sdk-migration` | **Date**: 2026-03-06 | **Spec**: `/home/adiazcan/github/claudian-copian/specs/001-copilot-sdk-migration/spec.md`
**Input**: Feature specification from `/specs/001-copilot-sdk-migration/spec.md`

**Note**: This plan is grounded in the GitHub Copilot SDK Node.js README and Authentication guide, plus the current Claudian Claude SDK integration surfaces.

## Summary

Replace the current Claude Code runtime with a GitHub Copilot SDK-backed provider adapter while keeping the Obsidian-facing chat, inline edit, context attachment, approval UX, and local conversation model as stable as possible. The technical approach is to concentrate change inside the current provider boundary: swap `query()`-based Claude orchestration for a `CopilotClient` plus `CopilotSession` lifecycle, normalize Copilot session events into Claudian’s existing stream model, replace Claude-specific CLI and auth readiness logic with Copilot readiness logic, and remove dependencies on Claude-native session files for primary workflows.

## Technical Context

**Language/Version**: TypeScript 5.x, Node.js 18+ runtime requirement from GitHub Copilot SDK, Obsidian desktop plugin environment  
**Primary Dependencies**: `obsidian`, `@github/copilot-sdk`, `@modelcontextprotocol/sdk`, `jest`, `ts-jest`, `esbuild`  
**Storage**: Vault-local JSON/JSONL files under `.claude/` for plugin data, plugin settings in Obsidian data storage, provider-native Copilot session state on local disk managed by Copilot CLI  
**Testing**: Jest unit and integration tests, TypeScript typecheck, ESLint, production build, manual Obsidian desktop smoke test  
**Target Platform**: Obsidian desktop on macOS, Linux, and Windows  
**Project Type**: Desktop plugin  
**Performance Goals**: Preserve streaming UX, avoid noticeable regressions in warm chat responsiveness, and keep cancellation, inline edit, and title generation responsive enough for interactive use  
**Constraints**: No dual-provider runtime, minimize fork drift from upstream Claudian, keep safety controls at least as strict as current behavior, depend on Copilot CLI plus supported authentication, and account for Copilot SDK technical preview instability  
**Scale/Scope**: Single-plugin provider replacement across core agent runtime, cold-start feature services, settings, tests, and provider-specific persistence assumptions

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

The repository constitution file is still an unfilled template, so there are no ratified constitutional gates to fail. Operational constraints come from repository guidance in `CLAUDE.md` instead:

- Keep changes minimal and concentrated in provider-specific surfaces.
- Prefer SDK-native capabilities over custom reimplementation when Copilot SDK already provides them.
- Follow TDD for new provider adapter behavior and bug fixes.
- Validate implementation with `npm run typecheck && npm run lint && npm run test && npm run build` before merge.

**Phase 0 gate result**: PASS

**Post-Phase 1 re-check**: PASS. The design keeps the provider swap concentrated in `src/core/agent`, `src/core/sdk`, a small number of feature services, and settings/readiness surfaces, avoiding broad architectural churn.

## Project Structure

### Documentation (this feature)

```text
specs/001-copilot-sdk-migration/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── provider-adapter.md
└── tasks.md
```

### Source Code (repository root)

```text
src/
├── main.ts
├── core/
│   ├── agent/
│   │   ├── ClaudianService.ts
│   │   ├── QueryOptionsBuilder.ts
│   │   ├── MessageChannel.ts
│   │   ├── SessionManager.ts
│   │   ├── customSpawn.ts
│   │   └── types.ts
│   ├── hooks/
│   │   └── SecurityHooks.ts
│   ├── sdk/
│   │   ├── transformSDKMessage.ts
│   │   ├── typeGuards.ts
│   │   └── types.ts
│   ├── security/
│   │   └── ApprovalManager.ts
│   ├── storage/
│   │   ├── SessionStorage.ts
│   │   ├── CCSettingsStorage.ts
│   │   └── ClaudianSettingsStorage.ts
│   └── types/
│       ├── settings.ts
│       ├── sdk.ts
│       ├── models.ts
│       └── chat.ts
├── features/
│   ├── chat/
│   │   ├── controllers/
│   │   └── services/
│   ├── inline-edit/
│   │   └── InlineEditService.ts
│   └── settings/
└── utils/
    ├── claudeCli.ts
    ├── env.ts
    ├── interrupt.ts
    ├── sdkSession.ts
    └── session.ts

tests/
├── __mocks__/
├── integration/
│   └── core/
├── unit/
│   ├── core/
│   ├── features/
│   └── utils/
└── helpers/
```

**Structure Decision**: Keep the single-project layout unchanged. Concentrate implementation work in existing provider-facing modules instead of introducing a new package or broad folder reorganization. The only new structural addition expected in source is a Copilot-specific adapter or helper surface inside existing `core/agent` and `core/sdk` areas.

## Complexity Tracking

No constitution violations require justification at planning time.
