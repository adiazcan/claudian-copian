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

*** Add File: /home/adiazcan/github/claudian-copian/specs/001-copilot-sdk-migration/research.md
# Research: GitHub Copilot Provider Replacement

## Decision 1: Use `CopilotClient` plus long-lived `CopilotSession` as the primary runtime abstraction

**Decision**: Replace Claude SDK `query()` orchestration with a shared `CopilotClient` and conversation-scoped `CopilotSession` handles for persistent chat, while still allowing short-lived sessions for cold-start feature services when appropriate.

**Rationale**: The Copilot SDK centers the runtime around a client that starts a CLI-backed server and sessions that send prompts, emit events, support resume, and can be disconnected independently. This maps more cleanly onto Claudian’s persistent chat model than emulating Claude’s per-query async iterator directly.

**Alternatives considered**:

- Emulate the old `query()` shape everywhere with an adapter that creates a new Copilot session per message. Rejected because it would fight the Copilot SDK session model and lose session-native features.
- Rebuild the entire conversation layer around raw Copilot events. Rejected because it would create unnecessary UI churn and merge drift.

## Decision 2: Normalize Copilot session events into Claudian’s existing stream model

**Decision**: Keep Claudian’s internal streaming contract and rewrite the provider transform layer to map Copilot events such as `assistant.message_delta`, `assistant.message`, `tool.execution_start`, `tool.execution_complete`, and `session.idle` into the existing UI-facing stream semantics.

**Rationale**: Most of the plugin UI already renders provider-agnostic chat chunks. Reusing that contract minimizes churn and limits change mostly to `core/sdk`, `core/agent`, and a few controllers.

**Alternatives considered**:

- Propagate Copilot event types directly into feature controllers and UI. Rejected because it would spread provider churn widely across the codebase.
- Keep only final-message rendering and drop streaming. Rejected because streaming continuity is a core product expectation.

## Decision 3: Default to signed-in user authentication for the Obsidian desktop plugin

**Decision**: Treat the normal desktop path as Copilot CLI login using the signed-in GitHub user, while still allowing explicit token-based auth through environment variables or constructor options for development, automation, and advanced setups.

**Rationale**: The Copilot auth guide explicitly describes signed-in user credentials as the default fit for interactive desktop apps. That matches an Obsidian plugin better than embedding a new OAuth app flow.

**Alternatives considered**:

- Build a custom OAuth GitHub App flow into the plugin. Rejected for higher scope, more maintenance, and no immediate need for a single-user desktop plugin.
- Require BYOK from the start. Rejected because the product goal is GitHub Copilot SDK replacement, not a provider abstraction redesign.

## Decision 4: Replace Claude pre-tool hooks with Copilot session hooks and explicit user-input handling

**Decision**: Rebuild approval, blocklist, and vault-boundary enforcement around Copilot session hooks (`onPreToolUse`, `onPostToolUse`, error hooks) and `onUserInputRequest` for agent-driven questions.

**Rationale**: The Copilot SDK already exposes hook points for tool gating and user input. Reusing those hooks preserves the current security and approval posture without keeping Claude SDK types or hook contracts.

**Alternatives considered**:

- Let Copilot tools run without plugin-side approval. Rejected because it weakens existing safety expectations.
- Reimplement tool execution outside the SDK. Rejected because the SDK already provides the control points needed.

## Decision 5: Decouple Claudian conversation persistence from provider-native Claude session files

**Decision**: Keep Claudian’s local conversation JSONL metadata and rendered message history, but stop depending on `~/.claude/projects/...` as a primary source of truth for session reconstruction.

**Rationale**: Copilot sessions are managed by Copilot CLI and expose explicit APIs for create, resume, list, delete, message retrieval, and disconnect. That means the plugin can rely on its own persisted conversation model plus provider session IDs rather than reading Claude-native on-disk JSONL structures.

**Alternatives considered**:

- Continue parsing provider-native session files from disk. Rejected because it keeps deep Claude assumptions in place and may not match Copilot storage semantics.
- Drop local Claudian conversation storage and use provider history only. Rejected because the plugin already has local metadata, previews, titles, and UI state built around its own storage layer.

## Decision 6: Remove Claude-specific CLI path and model/beta assumptions from the provider boundary

**Decision**: Replace Claude CLI path resolution, model enums, and beta-flag assumptions with Copilot CLI readiness checks, Copilot model identifiers, and Copilot-supported session config such as reasoning effort and infinite session settings.

**Rationale**: Copilot SDK uses a different CLI executable, a different model surface, and different advanced controls. Carrying forward Claude-specific fields would create confusion and fragile compatibility code.

**Alternatives considered**:

- Keep Claude naming in settings and translate invisibly. Rejected because it preserves the wrong mental model and makes future maintenance harder.
- Strip advanced model controls entirely. Rejected because the plugin already offers model selection and should keep a reasonable subset where Copilot supports it.

## Decision 7: Route cold-start features through a shared Copilot runtime helper instead of bespoke provider code

**Decision**: Refactor title generation, instruction refinement, and inline edit to use a shared Copilot runtime helper for session creation, prompt submission, streaming, cancellation, and readiness checks.

**Rationale**: These services currently duplicate Claude query setup. A shared helper reduces divergence and makes auth, CLI, and error handling consistent.

**Alternatives considered**:

- Migrate each feature service independently. Rejected because it duplicates provider setup logic in several places.
- Force every feature through the main persistent chat service. Rejected because some flows still benefit from isolated short-lived sessions.

## Decision 8: Explicitly classify unsupported Claude-exclusive capabilities before implementation begins

**Decision**: Create and maintain a capability compatibility record during implementation, with special attention to Claude-native session rewind/fork semantics, Claude plugin discovery, and any Claude-specific prompt or storage interop.

**Rationale**: The feature spec requires no silent removals. Some current behaviors are clearly Claude-shaped and need explicit disposition early so tasks do not assume parity that the Copilot SDK does not promise.

**Alternatives considered**:

- Defer incompatibility handling until integration testing. Rejected because it increases the chance of hidden scope changes late in the cycle.
- Promise full behavior parity up front. Rejected because the available Copilot docs do not justify that assumption.

## Decision 9: Treat Copilot SDK technical preview status as a design constraint

**Decision**: Keep the Copilot SDK usage wrapped behind a narrow adapter surface with focused tests so future SDK API changes affect a small number of modules.

**Rationale**: The Node.js README states the SDK is in technical preview and may change in breaking ways. A narrow boundary lowers future maintenance cost.

**Alternatives considered**:

- Use Copilot SDK types directly throughout the app. Rejected because it maximizes breakage when the SDK evolves.

*** Add File: /home/adiazcan/github/claudian-copian/specs/001-copilot-sdk-migration/data-model.md
# Data Model: GitHub Copilot Provider Replacement

## 1. Copilot Runtime Config

Represents the provider-specific runtime configuration used to start or connect the Copilot client.

**Fields**

- `cliPath`: optional explicit path to Copilot CLI executable
- `cliArgs`: optional additional CLI invocation arguments when needed
- `authSource`: one of `logged-in-user`, `explicit-token`, `environment-token`, `byok`
- `useLoggedInUser`: boolean
- `githubTokenPresent`: boolean
- `model`: selected Copilot model identifier
- `reasoningEffort`: optional reasoning level supported by the selected model
- `streamingEnabled`: boolean
- `infiniteSessionsEnabled`: boolean

**Validation Rules**

- `authSource` must resolve to one valid Copilot authentication path.
- `model` is required when using a custom provider path.
- CLI readiness must be validated before starting interactive workflows.

## 2. Provider Session Handle

Represents the live association between a Claudian conversation and a Copilot SDK session.

**Fields**

- `conversationId`: Claudian conversation identifier
- `providerSessionId`: Copilot session identifier
- `status`: one of `new`, `active`, `idle`, `aborting`, `disconnected`, `error`
- `workspacePath`: optional Copilot session workspace path when infinite sessions are enabled
- `model`: effective model for the session
- `startedAt`: timestamp
- `updatedAt`: timestamp
- `lastError`: optional provider-facing error summary

**Relationships**

- One Claudian conversation maps to zero or one active provider session handles at a time.
- A provider session handle may be resumed across multiple user turns.

**State Transitions**

- `new -> active` when the first prompt is sent
- `active -> idle` when Copilot emits `session.idle`
- `active -> aborting -> idle` when the user interrupts a turn
- `active|idle -> disconnected` on cleanup
- Any state -> `error` when provider startup or execution fails irrecoverably

## 3. Provider Event Envelope

Represents the normalized event payload sent from Copilot SDK surfaces into Claudian’s existing stream pipeline.

**Fields**

- `type`: normalized event kind such as `text`, `thinking`, `tool_use`, `tool_result`, `error`, `usage`, `session_init`, `turn_complete`
- `conversationId`: owning conversation
- `providerSessionId`: related Copilot session identifier
- `content`: optional text payload
- `toolName`: optional tool identifier
- `toolInput`: optional structured tool input
- `toolResult`: optional structured tool result
- `isError`: boolean
- `timestamp`: event time

**Validation Rules**

- Each envelope must map from one or more Copilot session events in a deterministic way.
- UI-facing controllers must not need raw Copilot event payloads for normal rendering.

## 4. Provider Readiness State

Represents the user-facing readiness of GitHub Copilot for this plugin instance.

**Fields**

- `cliAvailable`: boolean
- `authAvailable`: boolean
- `entitlementAvailable`: boolean or `unknown`
- `errorMessage`: optional recovery-friendly message
- `checkedAt`: timestamp

**Validation Rules**

- A failed readiness check must produce a user-facing explanation.
- Readiness must be checked before starting chat, inline edit, title generation, and instruction refinement.

## 5. Capability Compatibility Entry

Represents one previously Claude-dependent capability and its disposition under Copilot.

**Fields**

- `capabilityName`: human-readable identifier
- `area`: one of `chat`, `inline-edit`, `security`, `sessions`, `settings`, `plugins`, `mcp`, `commands`
- `status`: one of `preserved`, `changed`, `replaced`, `unsupported`
- `notes`: explanation of the behavior under Copilot
- `userVisibleImpact`: boolean

**Relationships**

- Many compatibility entries attach to one feature release.
- Entries inform settings cleanup, release notes, and regression testing.

## 6. Workspace Configuration

Represents persisted plugin configuration that survives the provider swap when it is provider-agnostic.

**Fields**

- `slashCommands`
- `agentDefinitions`
- `mcpServers`
- `permissionMode`
- `blockedCommands`
- `allowedExportPaths`
- `environmentVariables`
- `providerSettings`

**Validation Rules**

- Provider-agnostic settings should continue to load unchanged.
- Provider-specific fields must either be renamed, replaced, or removed with explicit handling.

*** Add File: /home/adiazcan/github/claudian-copian/specs/001-copilot-sdk-migration/contracts/provider-adapter.md
# Contract: Provider Adapter

## Purpose

Define the internal contract between Claudian’s feature layer and the GitHub Copilot SDK-backed runtime so the provider swap remains isolated.

## Responsibilities

The provider adapter is responsible for:

- starting and stopping the shared Copilot client
- checking provider readiness before interactive workflows
- creating, resuming, disconnecting, and aborting provider sessions
- sending prompts with attachments and contextual configuration
- translating Copilot session events into Claudian stream events
- applying approval, blocklist, and vault-boundary policy through Copilot hooks
- surfacing provider errors as user-facing recovery messages

## Required Operations

### `ensureClientReady()`

**Input**

- optional runtime overrides

**Output**

- readiness result containing CLI, auth, and recoverability status

**Behavior**

- validates Copilot CLI availability
- validates usable authentication according to the configured strategy
- does not start a user-visible workflow on failure

### `openSession(conversationId, config)`

**Input**

- Claudian conversation identifier
- model and reasoning options
- streaming flag
- tool and hook configuration

**Output**

- provider session handle

**Behavior**

- creates or resumes a Copilot session for the conversation
- returns a stable provider session identifier
- stores enough metadata for later reuse in the same conversation

### `sendMessage(sessionHandle, message)`

**Input**

- provider session handle
- prompt text
- optional attachments
- optional dispatch mode

**Output**

- queued message identifier

**Behavior**

- sends the prompt through the active Copilot session
- begins normalized event emission immediately when streaming is enabled

### `abortSessionTurn(sessionHandle)`

**Behavior**

- aborts the current in-flight turn if one exists
- results in a deterministic Claudian-side interrupted or idle state

### `disconnectSession(sessionHandle)`

**Behavior**

- disconnects runtime resources without corrupting persisted Claudian conversation state

## Event Normalization Contract

Copilot SDK events must be mapped into Claudian stream semantics as follows:

- `assistant.message_delta` -> incremental text chunk
- `assistant.reasoning_delta` -> incremental thinking chunk when supported
- `assistant.message` -> final assistant text event
- `assistant.reasoning` -> final thinking event when supported
- `tool.execution_start` -> tool use event
- `tool.execution_complete` -> tool result event
- `session.idle` -> turn completion signal
- session creation or resume metadata -> session initialization event
- provider execution errors -> normalized error event with recovery-friendly text

The feature and UI layers must not depend on raw Copilot event names.

## Approval And Tool Policy Contract

- Pre-tool decisions must still support allow, deny, and ask behavior.
- Vault confinement and export-path restrictions must remain enforceable before tool execution.
- Read-only inline edit restrictions must remain enforceable.
- Agent questions that require user input must route through a user-input callback rather than raw provider UI.

## Error Contract

The adapter must classify and surface at least these failure categories:

- CLI unavailable
- authentication unavailable
- entitlement unavailable
- connectivity or startup failure
- session creation failure
- turn execution failure
- tool policy denial

Each category must provide a user-facing message and an implementation-facing error code or discriminator suitable for tests.

*** Add File: /home/adiazcan/github/claudian-copian/specs/001-copilot-sdk-migration/quickstart.md
# Quickstart: GitHub Copilot Provider Replacement

## Goal

Implement and validate the replacement of Claude Code runtime usage with GitHub Copilot SDK while keeping Claudian’s Obsidian workflows stable.

## Prerequisites

- Node.js 18 or newer
- Obsidian desktop development environment
- GitHub Copilot CLI installed and available on `PATH`, or a known explicit CLI path
- A usable Copilot authentication path:
  - signed-in interactive user via Copilot CLI
  - explicit GitHub token for development or automation
  - environment-based token for non-interactive testing

## Implementation Sequence

1. Add the GitHub Copilot SDK dependency and introduce a provider adapter layer in the existing `src/core/agent` and `src/core/sdk` boundaries.
2. Replace persistent chat orchestration in `ClaudianService` with `CopilotClient` and `CopilotSession` lifecycle management.
3. Rewrite stream normalization so Copilot session events produce the same Claudian-facing chat chunks used by current controllers.
4. Rebuild approval, blocklist, and vault restriction logic using Copilot session hooks.
5. Migrate cold-start services such as inline edit, title generation, and instruction refinement to the shared Copilot runtime helper.
6. Replace Claude CLI readiness and settings surfaces with Copilot CLI readiness and Copilot-focused provider settings.
7. Remove dependencies on Claude-native session file parsing where those files are no longer the source of truth.
8. Update tests, mocks, and fixtures to use Copilot runtime shapes.

## Verification Commands

```bash
npm install
npm run typecheck
npm run lint
npm run test
npm run build
```

## Manual Smoke Tests

1. Launch the plugin in Obsidian with GitHub Copilot CLI installed and authenticated.
2. Open the chat sidebar and send a prompt with at least one attached vault file.
3. Confirm streaming output, tool activity, and completion behavior render correctly.
4. Trigger inline edit on selected text and verify the diff/apply flow still works.
5. Trigger instruction refinement and title generation to confirm short-lived Copilot-backed flows work.
6. Deny or block a tool action and verify approval and safety messaging still behaves correctly.
7. Test a missing-auth or missing-CLI scenario and verify the plugin shows clear recovery guidance.

## Exit Criteria

- Claude SDK is no longer required for primary workflows.
- Main chat, inline edit, instruction refinement, and title generation run through Copilot.
- Provider-specific changes remain concentrated in the documented adapter boundary.
- Unsupported Claude-only behaviors are explicitly documented before release.
