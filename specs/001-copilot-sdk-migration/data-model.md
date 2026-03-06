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
