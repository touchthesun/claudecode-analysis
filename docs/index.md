# Claude Code Source Map

This documentation set maps the `src/` codebase of Claude Code, including architecture, runtime flows, design patterns, and capability ownership.

## Recommended reading order

1. [Architecture Overview](architecture-overview.md)
2. [Runtime Flow](runtime-flow.md)
3. [Module Map](module-map.md)
4. [Design Patterns](design-patterns.md)
5. [Architecture Decisions](architecture-decisions.md)
6. [External Findings Cross-Check](external-findings-cross-check.md)
7. [Anti-Patterns System Design Review](anti-patterns-system-design-review.md)
8. [Key Capabilities](key-capabilities.md)
9. [Glossary](glossary.md)

## What this map covers

- Major subsystems and their boundaries
- Execution modes and startup/interactive runtime paths
- How query/tool orchestration works
- Notable design patterns and architectural tradeoffs
- Critical anti-patterns and "what not to do" examples
- Cross-checked notable external findings with source-backed validation
- Where key product capabilities are implemented

## Scope and method

- Source base: `src/`
- Evidence style: source-backed references to core entrypoints and subsystem files
- Documentation target: mkdocs-compatible Markdown in top-level `docs/`

## Primary source anchors

- `src/entrypoints/cli.tsx`
- `src/main.tsx`
- `src/entrypoints/init.ts`
- `src/replLauncher.tsx`
- `src/screens/REPL.tsx`
- `src/query.ts`
- `src/Tool.ts`
- `src/tools.ts`
- `src/commands.ts`
- `src/bootstrap/state.ts`
- `src/context.ts`
- `src/services/api/client.ts`
- `src/utils/plugins/pluginLoader.ts`
- `src/state/store.ts`
- `src/state/AppState.tsx`

## Next expansion ideas

- Add deeper per-module internals for highest-change areas (`query/`, `tools/`, `services/mcp/`).
- Add sequence-focused pages for permissions, compaction, and session restore.
- Add test-map pages once test structure is included in scope.
