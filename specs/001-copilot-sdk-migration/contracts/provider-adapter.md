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

## Capability Disposition Matrix

## Phase 1 Capability Checklist

### Claude-derived plugins

- current dependency assumptions
- Copilot replacement strategy
- user-visible impact notes

### Session fork and rewind semantics

- current dependency assumptions
- Copilot replacement strategy
- user-visible impact notes

### Subagent behavior

- current dependency assumptions
- Copilot replacement strategy
- user-visible impact notes

### MCP parity

- current dependency assumptions
- Copilot replacement strategy
- user-visible impact notes

### Provider-specific settings

- current dependency assumptions
- Copilot replacement strategy
- user-visible impact notes

### Session-file assumptions

- current dependency assumptions
- Copilot replacement strategy
- user-visible impact notes

| Capability | Current Claude Dependency | Copilot Outcome | User-Visible Impact | Owner Task |
|------------|---------------------------|-----------------|---------------------|------------|
| Claude plugin discovery | Claude plugin directory and enablement model | Unsupported or replaced | Yes | T009 / T037 |
| Session fork and rewind | Claude session `resumeAt` and fork semantics | Needs explicit decision | Yes | T009 / T037 |
| Subagent behavior | Claude task and subagent event model | Needs explicit decision | Yes | T009 / T037 |
| MCP activation and storage | Existing MCP manager and storage integration | Preserve if provider-neutral | Yes | T035 / T037 |
| External context attachment | Existing context mention and attachment flow | Preserve | Yes | T034 |
| Agent definition loading | Existing agent manager and storage | Preserve if provider-neutral | Yes | T036 |
| Claude-specific settings | Claude CLI path and related settings | Replace | Yes | T009 / T042 |
| Claude session file parsing | Claude native on-disk session format | Remove | No | T009 / T017 |
