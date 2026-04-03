# Key Capabilities

This page maps major Claude Code capabilities to their implementation anchors in `src/`.

## CLI and runtime mode dispatch

- Bootstrapped mode selection and fast paths:
  - `src/entrypoints/cli.tsx`
- Main command-line orchestration and startup path:
  - `src/main.tsx`
  - `src/entrypoints/init.ts`

## Interactive REPL experience

- REPL mounting and app wrapper:
  - `src/replLauncher.tsx`
  - `src/components/App.js` (loaded from launcher)
- Interactive shell and conversation UX:
  - `src/screens/REPL.tsx`
  - `src/components/*`
  - `src/ink/*`

## Query orchestration and streaming

- Turn loop, message normalization, tool/result iterations, compaction/token budget:
  - `src/query.ts`
  - `src/query/*`
- Message shaping utilities:
  - `src/utils/messages.ts`
  - `src/types/message.ts`

## Tooling system

- Tool contracts and execution context:
  - `src/Tool.ts`
- Tool registry/composition:
  - `src/tools.ts`
- Built-in tool implementations:
  - `src/tools/*`

## Command system

- Command catalog and enablement:
  - `src/commands.ts`
- Individual command implementations:
  - `src/commands/*`

## Agent and teammate workflows

- Agent tooling and definitions:
  - `src/tools/AgentTool/*`
- Team/swarm infrastructure and backend execution:
  - `src/utils/swarm/*`
  - `src/utils/swarm/backends/registry.ts`
- Task primitives:
  - `src/tasks/*`
  - `src/Task.ts`
  - `src/tasks.ts`

## Plugin and skills extensibility

- Plugin loading/discovery/validation:
  - `src/utils/plugins/pluginLoader.ts`
  - `src/utils/plugins/*`
- Bundled plugins:
  - `src/plugins/*`
- Skills:
  - `src/skills/*`
  - `src/utils/skills/*`

## MCP support

- MCP entrypoint and integration:
  - `src/entrypoints/mcp.ts`
  - `src/services/mcp/*`
- MCP-facing tools:
  - `src/tools/ListMcpResourcesTool/*`
  - `src/tools/ReadMcpResourceTool/*`

## API provider support (Anthropic + cloud providers)

- API client construction and auth wiring:
  - `src/services/api/client.ts`
- Provider and model helper logic:
  - `src/utils/model/providers.ts`
  - `src/utils/model/model.ts`
  - `src/utils/auth.ts`

## Policy, settings, and permissions

- Policy limits:
  - `src/services/policyLimits/*`
- Remote managed settings:
  - `src/services/remoteManagedSettings/*`
- Local settings and validation:
  - `src/utils/settings/*`
- Permission setup and enforcement helpers:
  - `src/utils/permissions/*`

## Session persistence and recovery

- Session storage and transcript utilities:
  - `src/utils/sessionStorage.ts`
  - `src/utils/conversationRecovery.ts`
  - `src/utils/sessionRestore.ts`
- Background/concurrent session handling:
  - `src/utils/concurrentSessions.ts`
  - `src/cli/bg.js` (fast-path dispatch from CLI)

## Telemetry and diagnostics

- Startup and query profiling helpers:
  - `src/utils/startupProfiler.ts`
  - `src/utils/queryProfiler.ts`
- Analytics and event logging:
  - `src/services/analytics/*`
- Telemetry helper modules:
  - `src/utils/telemetry/*`

## Remote-control and specialized runtime surfaces

- Bridge/remote-control:
  - `src/bridge/*`
- Daemon mode:
  - `src/daemon/*`
- Environment/self-hosted runners:
  - `src/environment-runner/*`
  - `src/self-hosted-runner/*`
- Server/direct-connect paths:
  - `src/server/*`
  - `src/remote/*`

## Context enrichment and repository awareness

- System/user context assembly:
  - `src/context.ts`
- Git and CLAUDE.md memory retrieval:
  - `src/utils/git.ts`
  - `src/utils/claudemd.ts`

## Notable internally-implemented safeguards and behaviors

- Undercover mode and attribution masking behavior:
  - `src/utils/undercover.ts`
  - `src/utils/attribution.ts`
  - `src/commands/commit.ts`
  - `src/commands/commit-push-pr.ts`
- Prompt-level anti-distillation/first-party beta behavior:
  - `src/services/api/claude.ts`
  - `src/utils/betas.ts`
  - `src/constants/betas.ts`
- Request attribution and native attestation placeholder integration:
  - `src/constants/system.ts`
  - `src/utils/http.ts`
- Auto-compact resilience circuit breaker:
  - `src/services/compact/autoCompact.ts`
- Cron/scheduled background scaffolding (feature-gated):
  - `src/utils/cronTasks.ts`
  - `src/utils/cronScheduler.ts`
  - `src/tools/ScheduleCronTool/prompt.ts`
- Frustration keyword detection:
  - `src/utils/userPromptKeywords.ts`

## Related pages

- [Module Map](module-map.md)
- [Runtime Flow](runtime-flow.md)
- [Glossary](glossary.md)
- [External Findings Cross-Check](external-findings-cross-check.md)
