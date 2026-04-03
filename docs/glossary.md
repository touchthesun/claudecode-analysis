# Glossary

## Agent

A model-driven execution unit that can run queries and tools, including nested/background contexts. Core agent behavior is routed through `query.ts`, `Tool.ts`, and `tools/AgentTool/*`.

## App State

The interactive UI/runtime state managed through a custom store and React context (`state/store.ts`, `state/AppState.tsx`).

## Bootstrap State

Process-level session and telemetry state in `bootstrap/state.ts`, shared across runtime layers.

## Command

A slash or prompt-level operation registered in `commands.ts` and implemented in `commands/*`.

## Context (System/User)

Ambient prompt data injected per turn from `context.ts`, including git snapshot, CLAUDE.md content, and date metadata.

## Fast Path

A specialized CLI dispatch route in `entrypoints/cli.tsx` that avoids loading full runtime for mode-specific operations.

## Feature Gate

A compile-time/runtime conditional (`feature('...')` and env guards) used to include/exclude capabilities by build or environment.

## MCP

Model Context Protocol integration layer for external tools/resources, implemented in `services/mcp/*` and exposed through MCP-specific tools.

## Plugin

An extension package with optional manifest and structured directories for commands/agents/hooks, loaded by `utils/plugins/pluginLoader.ts`.

## Query Loop

The iterative assistant-tool orchestration loop in `query.ts` that streams responses, executes tools, and continues until terminal conditions.

## REPL

The interactive terminal experience coordinated in `screens/REPL.tsx`, rendered via Ink and app components.

## Tool

A model-invokable capability defined by the `Tool` contract (`Tool.ts`) and composed in `tools.ts` with implementation under `tools/*`.

## ToolUseContext

The typed execution context object passed into tools, containing app state handles, permission context, MCP clients/resources, and runtime controls.

## Worktree Mode

A workflow mode integrating git worktrees (and optionally tmux), surfaced via CLI and helper utilities like `utils/worktree.ts`.

## Related pages

- [Architecture Overview](architecture-overview.md)
- [Design Patterns](design-patterns.md)
- [Key Capabilities](key-capabilities.md)
