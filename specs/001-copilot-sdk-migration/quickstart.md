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