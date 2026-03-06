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
