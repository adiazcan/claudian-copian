# Tasks: GitHub Copilot Provider Replacement

**Input**: Design documents from `/specs/001-copilot-sdk-migration/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/, quickstart.md

**Tests**: Tests are required for this feature because repository guidance mandates TDD for new provider adapter behavior and bug-fix level runtime changes.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g. [US1], [US2], [US3])
- Include exact file paths in descriptions

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Introduce the Copilot SDK dependency and create the planning/documentation anchors the implementation will follow.

- [ ] T001 Update dependencies from Claude SDK to GitHub Copilot SDK in /home/adiazcan/github/claudian-copian/package.json
- [ ] T002 [P] Replace the Claude SDK mock with a Copilot SDK mock surface in /home/adiazcan/github/claudian-copian/tests/__mocks__/claude-agent-sdk.ts
- [ ] T003 [P] Add or rename provider-facing helper fixtures for Copilot session events in /home/adiazcan/github/claudian-copian/tests/helpers/sdkMessages.ts
- [ ] T004 [P] Create the initial provider-boundary skeleton and capability checklist headings in /home/adiazcan/github/claudian-copian/specs/001-copilot-sdk-migration/contracts/provider-adapter.md

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Create the Copilot runtime boundary, event normalization, provider settings/readiness infrastructure, and compatibility inventory that all user stories depend on.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T005 [P] Add unit tests for Copilot provider type mappings and normalized event types in /home/adiazcan/github/claudian-copian/tests/unit/core/agent/types.test.ts
- [ ] T006 [P] Add unit tests for Copilot session and readiness state handling in /home/adiazcan/github/claudian-copian/tests/unit/core/agent/SessionManager.test.ts
- [ ] T007 [P] Add unit tests for Copilot option and hook building in /home/adiazcan/github/claudian-copian/tests/unit/core/agent/QueryOptionsBuilder.test.ts
- [ ] T008 Define Copilot-facing provider types and adapter interfaces in /home/adiazcan/github/claudian-copian/src/core/agent/types.ts
- [ ] T009 Create a capability compatibility inventory covering Claude-derived plugins, fork and rewind semantics, subagents, MCP parity, provider-specific settings, and session-file assumptions in /home/adiazcan/github/claudian-copian/specs/001-copilot-sdk-migration/contracts/provider-adapter.md
- [ ] T010 Implement Copilot client/session configuration building in /home/adiazcan/github/claudian-copian/src/core/agent/QueryOptionsBuilder.ts
- [ ] T011 Implement shared Copilot session lifecycle tracking in /home/adiazcan/github/claudian-copian/src/core/agent/SessionManager.ts
- [ ] T012 Implement normalized Copilot event transformation in /home/adiazcan/github/claudian-copian/src/core/sdk/transformSDKMessage.ts
- [ ] T013 Update provider type exports and guards for Copilot runtime usage in /home/adiazcan/github/claudian-copian/src/core/types/sdk.ts and /home/adiazcan/github/claudian-copian/src/core/sdk/typeGuards.ts
- [ ] T014 Replace Claude-specific model and provider settings types with Copilot-oriented settings in /home/adiazcan/github/claudian-copian/src/core/types/models.ts and /home/adiazcan/github/claudian-copian/src/core/types/settings.ts
- [ ] T015 Implement Copilot CLI readiness and provider auth detection in /home/adiazcan/github/claudian-copian/src/utils/claudeCli.ts and /home/adiazcan/github/claudian-copian/src/utils/env.ts
- [ ] T016 Rework approval and vault-boundary enforcement onto Copilot hooks in /home/adiazcan/github/claudian-copian/src/core/hooks/SecurityHooks.ts and /home/adiazcan/github/claudian-copian/src/core/security/ApprovalManager.ts
- [ ] T017 Remove Claude-native session file assumptions from provider utilities in /home/adiazcan/github/claudian-copian/src/utils/sdkSession.ts and /home/adiazcan/github/claudian-copian/src/utils/session.ts

**Checkpoint**: Foundation ready. The codebase has a Copilot adapter boundary, normalized event model, provider readiness layer, and an explicit compatibility inventory.

---

## Phase 3: User Story 1 - Continue Core AI Workflows (Priority: P1) 🎯 MVP

**Goal**: Replace the Claude runtime in chat and inline edit while preserving streaming, attachments, tool events, and end-user workflow continuity.

**Independent Test**: Open the chat sidebar, send a file-aware prompt through GitHub Copilot, verify streamed output and completion, then run inline edit on selected text and confirm the diff/apply flow works.

### Tests for User Story 1 ⚠️

> **NOTE: Write these tests first, ensure they fail before implementation.**

- [ ] T018 [P] [US1] Update core chat runtime unit tests for Copilot session flow in /home/adiazcan/github/claudian-copian/tests/unit/core/agent/ClaudianService.test.ts
- [ ] T019 [P] [US1] Update message queue and turn-handling tests for Copilot message semantics in /home/adiazcan/github/claudian-copian/tests/unit/core/agent/MessageChannel.test.ts
- [ ] T020 [P] [US1] Add or update integration coverage for end-to-end Copilot chat behavior in /home/adiazcan/github/claudian-copian/tests/integration/core/agent/ClaudianService.test.ts
- [ ] T021 [P] [US1] Update inline edit unit coverage for Copilot-backed read-only tool flow in /home/adiazcan/github/claudian-copian/tests/unit/features/inline-edit/InlineEditService.test.ts

### Implementation for User Story 1

- [ ] T022 [US1] Replace the persistent Claude query runtime with Copilot client/session orchestration in /home/adiazcan/github/claudian-copian/src/core/agent/ClaudianService.ts
- [ ] T023 [US1] Update message queueing and turn state handling for Copilot event delivery in /home/adiazcan/github/claudian-copian/src/core/agent/MessageChannel.ts
- [ ] T024 [US1] Adapt chat streaming controller logic to the normalized Copilot event model in /home/adiazcan/github/claudian-copian/src/features/chat/controllers/StreamController.ts and /home/adiazcan/github/claudian-copian/src/features/chat/controllers/InputController.ts
- [ ] T025 [US1] Replace Claude interrupt and completion heuristics with Copilot session abort semantics in /home/adiazcan/github/claudian-copian/src/utils/interrupt.ts
- [ ] T026 [US1] Migrate inline edit execution to the shared Copilot runtime and Copilot hook model in /home/adiazcan/github/claudian-copian/src/features/inline-edit/InlineEditService.ts
- [ ] T027 [US1] Update chat session metadata handling for Copilot provider session IDs in /home/adiazcan/github/claudian-copian/src/core/storage/SessionStorage.ts and /home/adiazcan/github/claudian-copian/src/core/types/chat.ts

**Checkpoint**: User Story 1 is fully functional and testable independently with GitHub Copilot.

---

## Phase 4: User Story 2 - Preserve Product Behavior With Minimal Fork Drift (Priority: P2)

**Goal**: Isolate provider-specific logic so upstream Claudian merges mainly touch the provider boundary instead of broad feature and UI code, while preserving provider-agnostic workflows.

**Independent Test**: Review the changed file set and confirm provider-specific logic is concentrated in the documented adapter boundary, with feature-facing flows using normalized interfaces rather than raw Copilot SDK details.

### Tests for User Story 2 ⚠️

- [ ] T028 [P] [US2] Add unit tests covering Copilot adapter boundaries and unsupported capability classification in /home/adiazcan/github/claudian-copian/tests/unit/core/agent/index.test.ts
- [ ] T029 [P] [US2] Add regression tests for provider-agnostic session persistence and metadata handling in /home/adiazcan/github/claudian-copian/tests/unit/core/storage/SessionStorage.test.ts

### Implementation for User Story 2

- [ ] T030 [US2] Introduce or refine a narrow provider adapter export surface in /home/adiazcan/github/claudian-copian/src/core/agent/index.ts and /home/adiazcan/github/claudian-copian/src/core/sdk/index.ts
- [ ] T031 [US2] Remove Claude-specific spawn and executable assumptions from provider startup code in /home/adiazcan/github/claudian-copian/src/core/agent/customSpawn.ts and /home/adiazcan/github/claudian-copian/src/main.ts
- [ ] T032 [US2] Separate provider-agnostic storage and workflow settings from provider-specific fields in /home/adiazcan/github/claudian-copian/src/core/storage/ClaudianSettingsStorage.ts and /home/adiazcan/github/claudian-copian/src/core/storage/CCSettingsStorage.ts
- [ ] T033 [US2] Update plugin and slash-command integration points to consume provider-neutral runtime behavior in /home/adiazcan/github/claudian-copian/src/core/commands/builtInCommands.ts and /home/adiazcan/github/claudian-copian/src/core/types/settings.ts
- [ ] T034 [US2] Preserve external context attachment and mention-resolution behavior in /home/adiazcan/github/claudian-copian/src/features/chat/controllers/InputController.ts, /home/adiazcan/github/claudian-copian/src/utils/contextMentionResolver.ts, /home/adiazcan/github/claudian-copian/src/utils/externalContext.ts, and /home/adiazcan/github/claudian-copian/src/utils/externalContextScanner.ts
- [ ] T035 [US2] Preserve MCP activation, storage compatibility, and settings behavior in /home/adiazcan/github/claudian-copian/src/core/mcp/McpServerManager.ts, /home/adiazcan/github/claudian-copian/src/core/storage/McpStorage.ts, and /home/adiazcan/github/claudian-copian/src/features/settings/ui/McpSettingsManager.ts
- [ ] T036 [US2] Preserve provider-agnostic agent definition loading and settings behavior in /home/adiazcan/github/claudian-copian/src/core/agents/AgentManager.ts, /home/adiazcan/github/claudian-copian/src/core/agents/AgentStorage.ts, and /home/adiazcan/github/claudian-copian/src/features/settings/ui/AgentSettings.ts
- [ ] T037 [US2] Finalize the compatibility matrix with preserved, changed, replaced, and unsupported capabilities in /home/adiazcan/github/claudian-copian/specs/001-copilot-sdk-migration/contracts/provider-adapter.md and /home/adiazcan/github/claudian-copian/specs/001-copilot-sdk-migration/quickstart.md

**Checkpoint**: The provider swap is isolated enough that core product behavior remains stable and future merges primarily affect the provider boundary.

---

## Phase 5: User Story 3 - Configure And Operate GitHub Copilot Clearly (Priority: P3)

**Goal**: Make Copilot CLI readiness, authentication, first-run setup, and provider failures understandable from within the plugin.

**Independent Test**: Configure the plugin in a desktop environment, verify the success path to first response, then simulate missing CLI or auth and confirm the UI shows clear recovery guidance.

### Tests for User Story 3 ⚠️

- [ ] T038 [P] [US3] Add unit tests for Copilot readiness and failure messaging in /home/adiazcan/github/claudian-copian/tests/unit/features/chat/services/InstructionRefineService.test.ts and /home/adiazcan/github/claudian-copian/tests/unit/features/chat/services/TitleGenerationService.test.ts
- [ ] T039 [P] [US3] Create or extend settings regression coverage for Copilot provider configuration in /home/adiazcan/github/claudian-copian/tests/unit/features/settings/ClaudianSettings.test.ts
- [ ] T040 [P] [US3] Add explicit regression coverage for the default signed-in-user Copilot auth path in /home/adiazcan/github/claudian-copian/tests/unit/core/agent/ClaudianService.test.ts or /home/adiazcan/github/claudian-copian/tests/integration/core/agent/ClaudianService.test.ts

### Implementation for User Story 3

- [ ] T041 [US3] Replace cold-start title generation and instruction refinement runtime setup with Copilot-backed helpers in /home/adiazcan/github/claudian-copian/src/features/chat/services/TitleGenerationService.ts and /home/adiazcan/github/claudian-copian/src/features/chat/services/InstructionRefineService.ts
- [ ] T042 [US3] Update settings defaults and persisted provider fields for Copilot CLI and auth behavior in /home/adiazcan/github/claudian-copian/src/core/types/settings.ts and /home/adiazcan/github/claudian-copian/src/core/storage/ClaudianSettingsStorage.ts
- [ ] T043 [US3] Update settings UI labels, provider configuration controls, and readiness messaging in /home/adiazcan/github/claudian-copian/src/features/settings/ClaudianSettings.ts and /home/adiazcan/github/claudian-copian/src/features/settings/ui/EnvSnippetManager.ts
- [ ] T044 [US3] Update provider notices and unsupported-feature messaging across all maintained locale files under /home/adiazcan/github/claudian-copian/src/i18n/locales and the related settings UI in /home/adiazcan/github/claudian-copian/src/features/settings/ui/PluginSettingsManager.ts
- [ ] T045 [US3] Refresh repository-level user guidance for Copilot CLI setup and authentication in /home/adiazcan/github/claudian-copian/README.md and /home/adiazcan/github/claudian-copian/manifest.json

**Checkpoint**: Users can identify how to enable Copilot, how the plugin uses it, and how to recover from common provider failures.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final cleanup, compatibility hardening, and full validation across stories.

- [ ] T046 [P] Update or replace remaining Claude-specific mocks, fixtures, and helper references across /home/adiazcan/github/claudian-copian/tests/__mocks__ and /home/adiazcan/github/claudian-copian/tests/helpers
- [ ] T047 Clean up obsolete Claude-only utilities and dead code in /home/adiazcan/github/claudian-copian/src/utils/claudeCli.ts and /home/adiazcan/github/claudian-copian/src/core/plugins/PluginManager.ts
- [ ] T048 [P] Add final unit regressions for any uncovered Copilot event, auth, or tool-policy edge cases in /home/adiazcan/github/claudian-copian/tests/unit/core and /home/adiazcan/github/claudian-copian/tests/unit/features
- [ ] T049 Run quickstart validation steps and update any drift in /home/adiazcan/github/claudian-copian/specs/001-copilot-sdk-migration/quickstart.md
- [ ] T050 Run repository validation commands and fix any Copilot migration regressions surfaced by `npm run typecheck`, `npm run lint`, `npm run test`, and `npm run build`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies. Can start immediately.
- **Foundational (Phase 2)**: Depends on Setup completion. Blocks all user stories.
- **User Story 1 (Phase 3)**: Depends on Foundational completion.
- **User Story 2 (Phase 4)**: Depends on Foundational completion and is lower risk after User Story 1 proves the runtime path.
- **User Story 3 (Phase 5)**: Depends on Foundational completion and benefits from the runtime and readiness work already introduced.
- **Polish (Phase 6)**: Depends on all desired user stories being complete.

### User Story Dependencies

- **User Story 1 (P1)**: Starts after Phase 2. It is the MVP and establishes the end-to-end Copilot runtime path.
- **User Story 2 (P2)**: Starts after Phase 2 and preserves mergeability plus provider-agnostic workflows.
- **User Story 3 (P3)**: Starts after Phase 2 and finalizes setup, readiness, and user guidance.

### Within Each User Story

- Tests must be written and fail before implementation.
- Provider types and adapters before workflow wiring.
- Runtime wiring before feature-specific UX updates.
- Story documentation and compatibility notes before story sign-off.

### Parallel Opportunities

- T002, T003, and T004 can run in parallel.
- T005, T006, and T007 can run in parallel before foundational implementation.
- T018 through T021 can run in parallel as the US1 failing-test batch.
- T028 and T029 can run in parallel for US2.
- T038, T039, and T040 can run in parallel for US3.
- T046 and T048 can run in parallel during final polish.

---

## Parallel Example: User Story 1

```bash
# Launch the US1 failing-test batch together:
Task: "Update core chat runtime unit tests for Copilot session flow in tests/unit/core/agent/ClaudianService.test.ts"
Task: "Update message queue and turn-handling tests for Copilot message semantics in tests/unit/core/agent/MessageChannel.test.ts"
Task: "Add or update integration coverage for end-to-end Copilot chat behavior in tests/integration/core/agent/ClaudianService.test.ts"
Task: "Update inline edit unit coverage for Copilot-backed read-only tool flow in tests/unit/features/inline-edit/InlineEditService.test.ts"
```

---

## Parallel Example: User Story 3

```bash
# Launch the US3 failing-test batch together:
Task: "Add unit tests for Copilot readiness and failure messaging in tests/unit/features/chat/services/InstructionRefineService.test.ts and tests/unit/features/chat/services/TitleGenerationService.test.ts"
Task: "Create or extend settings regression coverage for Copilot provider configuration in tests/unit/features/settings/ClaudianSettings.test.ts"
Task: "Add explicit regression coverage for the default signed-in-user Copilot auth path in tests/unit/core/agent/ClaudianService.test.ts or tests/integration/core/agent/ClaudianService.test.ts"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational provider boundary.
3. Complete Phase 3: User Story 1.
4. Stop and validate chat and inline edit with GitHub Copilot before broadening scope.

### Incremental Delivery

1. Finish Setup and Foundational work to establish the Copilot runtime.
2. Deliver User Story 1 as the first working Copilot-backed release candidate.
3. Add User Story 2 to reduce merge drift and classify unsupported behavior.
4. Add User Story 3 to finalize provider readiness, settings, and user guidance.
5. Finish with full validation and cleanup.

### Parallel Team Strategy

1. One developer handles core runtime and event normalization.
2. One developer handles feature-service migration and settings/readiness.
3. One developer handles tests, mocks, and compatibility documentation.

---

## Notes

- [P] tasks are safe to parallelize because they target separate files or documentation.
- Story labels map every implementation task directly to a user story for traceability.
- The task list intentionally keeps provider churn concentrated in `src/core/agent`, `src/core/sdk`, a few feature services, and settings/readiness surfaces.
- Claude-only cleanup is deferred until the relevant Copilot replacement is already in place.