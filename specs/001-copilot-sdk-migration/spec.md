# Feature Specification: GitHub Copilot Provider Replacement

**Feature Branch**: `001-copilot-sdk-migration`  
**Created**: 2026-03-06  
**Status**: Draft  
**Input**: User description: "Change the current Claude Code integration to use GitHub Copilot SDK while preserving as much of the existing core architecture as possible, minimizing divergence from upstream and future merge conflicts. The project is an Obsidian plugin, we do not need to migrate users or maintain the Claude Code implementation. We want to move it to GitHub Copilot SDK."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Continue Core AI Workflows (Priority: P1)

As an Obsidian user, I want chat and inline editing to work through GitHub Copilot so I can keep using the plugin for file-aware, agentic work inside my vault.

**Why this priority**: The product loses its primary value if the main chat and editing workflows do not work after the provider replacement.

**Independent Test**: Can be fully tested by opening the chat sidebar, sending a prompt with vault context, and performing an inline edit request using GitHub Copilot, with successful streamed output and final results.

**Acceptance Scenarios**:

1. **Given** a user has configured GitHub Copilot access, **When** they send a normal chat request from the sidebar, **Then** the assistant responds in the existing plugin interface with streamed progress and a final answer.
2. **Given** a user selects text in a note, **When** they trigger inline edit, **Then** the request is processed through GitHub Copilot and the user can review and apply the resulting change.
3. **Given** a user includes vault files, editor selection, or supported external context in a request, **When** the request is sent, **Then** those inputs are preserved in the provider transition and available to the assistant.

---

### User Story 2 - Preserve Product Behavior With Minimal Fork Drift (Priority: P2)

As a maintainer, I want the provider-specific changes to stay isolated so I can keep pulling upstream Claudian updates with minimal merge conflict risk.

**Why this priority**: The user explicitly wants to keep the core architecture aligned with origin instead of creating a deep, hard-to-maintain fork.

**Independent Test**: Can be fully tested by reviewing the changed integration boundaries for a release update and confirming that unrelated core features, UI flows, and storage behavior remain unchanged unless they depend directly on the provider switch.

**Acceptance Scenarios**:

1. **Given** an upstream Claudian update that does not change provider-specific behavior, **When** the maintainer merges it, **Then** the merge requires changes only in the provider integration surface or directly affected feature areas.
2. **Given** an existing core capability such as slash commands, MCP activation, sessions, or security approval flows, **When** the provider migration is complete, **Then** that capability continues to behave the same unless the new provider cannot support it.
3. **Given** a feature cannot be preserved exactly under GitHub Copilot, **When** the maintainer reviews the replacement outcome, **Then** the difference is explicitly documented and exposed to users instead of silently failing.

---

### User Story 3 - Configure And Operate GitHub Copilot Clearly (Priority: P3)

As an Obsidian user, I want provider setup and failures to be clear so I can start using the plugin with GitHub Copilot without trial-and-error.

**Why this priority**: Replacing the provider only succeeds if users can understand how to enable the new provider and recover from common failures.

**Independent Test**: Can be fully tested by configuring the plugin on a supported desktop environment, confirming that the user can identify required setup steps, and verifying that missing sign-in or entitlement errors return clear recovery guidance.

**Acceptance Scenarios**:

1. **Given** the plugin is installed but GitHub Copilot is not ready to use, **When** the user opens the relevant workflow, **Then** the plugin explains what prerequisite is missing and how to resolve it.
2. **Given** GitHub Copilot access is available, **When** the user uses the plugin for the first time, **Then** the path from configuration to first successful response is clear and does not require source-level knowledge.
3. **Given** a request fails for a provider-related reason, **When** the failure is shown, **Then** the plugin presents a user-facing explanation and suggested next step.

### Edge Cases

- GitHub Copilot is unavailable because the user is not signed in or does not have the required entitlement.
- A request depends on a provider capability that GitHub Copilot cannot expose in the same way.
- A feature area currently assumes provider-specific tool or event behavior that does not exist under GitHub Copilot.
- The plugin is used in a desktop environment where provider access is partially configured but not fully operational.
- A user invokes advanced workflows such as slash commands, MCP-backed requests, or inline edit before provider setup is complete.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST use GitHub Copilot as the primary assistant provider for new chat and inline-edit interactions.
- **FR-002**: The system MUST preserve the existing Claudian user entry points for chat, inline edit, session management, slash commands, context attachment, and approval-driven workflows unless a capability is explicitly unsupported by the new provider.
- **FR-003**: The system MUST isolate provider-dependent behavior behind a narrowly scoped integration surface so that core product logic can remain aligned with upstream Claudian.
- **FR-004**: The system MUST identify and document each existing Claude-dependent capability as one of: preserved, behaviorally changed, replaced, or unsupported under GitHub Copilot.
- **FR-005**: The system MUST preserve supported plugin assets and workflows that are provider-agnostic, including reusable commands, agent definitions, MCP server configuration, and workspace-level settings.
- **FR-006**: The system MUST remove Claude Code as an active runtime dependency for primary workflows once the replacement is complete.
- **FR-007**: The system MUST fail gracefully when GitHub Copilot authentication, entitlement, or connectivity is unavailable, including guidance that lets the user recover without inspecting logs or source code.
- **FR-008**: The system MUST keep user safety controls, approval expectations, and vault boundary protections at least as restrictive as they are before the replacement unless the user explicitly changes those settings.
- **FR-009**: The system MUST preserve user-visible continuity for streaming responses, error handling, and task completion feedback across supported workflows.
- **FR-010**: The system MUST provide maintainers with a concise compatibility note that defines the intended provider-specific change boundary and lists known unsupported or behaviorally changed areas.
- **FR-011**: The system MUST present unsupported features and provider limitations explicitly to users rather than removing them silently.
- **FR-012**: The system MUST not require the project to continue shipping or maintaining Claude Code as an alternate provider path.

### Key Entities *(include if feature involves data)*

- **Provider Integration Surface**: The bounded set of behaviors where Claudian communicates with the external assistant provider, including request submission, streaming events, tool permissions, and provider-specific errors.
- **Workspace Configuration**: The collection of persisted user settings, command definitions, agent definitions, MCP server entries, and provider-related preferences that must remain usable after the replacement where they are provider-agnostic.
- **Capability Compatibility Record**: The documented mapping of current Claude-based behaviors to their GitHub Copilot outcome: preserved, changed, replaced, or unsupported.
- **Conversation Session**: A stored interaction history and metadata set used by the plugin while operating through GitHub Copilot.

## Assumptions

- Users who want the migrated experience can authenticate to GitHub Copilot outside the plugin through the normal account flow supported on their machine.
- The goal is continuity of user-facing Claudian behavior, not exact preservation of Claude-specific internal behavior.
- Some Claude-exclusive features may need to be retired or replaced if GitHub Copilot does not expose an equivalent capability.
- Upstream Claudian will continue evolving, so the local customization should stay concentrated in clearly defined provider-specific areas rather than broad rewrites.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can complete the two primary workflows, chat and inline edit, end-to-end with GitHub Copilot in under 3 minutes each on a configured vault.
- **SC-002**: At least 90% of supported provider-agnostic plugin workflows remain available after the provider replacement without requiring redesign of the surrounding user experience.
- **SC-003**: For provider-related failures such as missing sign-in, missing entitlement, or unavailable connectivity, 100% of tested cases return user-facing recovery guidance instead of a silent failure or raw crash.
- **SC-004**: During routine upstream syncs that do not intentionally change provider behavior, maintainers can complete the merge by modifying only the documented provider divergence area and directly affected feature surfaces in at least 80% of sampled updates.
- **SC-005**: All Claude-dependent capabilities present before replacement are classified and communicated to users or maintainers before release, with zero untracked behavior removals discovered during release review.
