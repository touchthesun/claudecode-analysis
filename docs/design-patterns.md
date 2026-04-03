# Design Patterns

This page captures recurring implementation patterns in `src/`, with concrete source anchors.

## 1) Composition root registries

Claude Code uses central registries to compose runtime-visible capabilities.

- `src/tools.ts`:
  - `getAllBaseTools()` is the canonical tool catalog.
  - `getTools()` filters by environment, feature flags, and permission rules.
- `src/commands.ts`:
  - `COMMANDS` memoized list composes built-in command modules and conditional commands.

Why it matters:
- Keeps capability exposure explicit and discoverable.
- Makes mode/policy gating easier to audit in one location.

## 2) Strategy + runtime backend selection

Swarm pane execution uses a strategy-style backend abstraction:

- `src/utils/swarm/backends/types.ts` defines backend contracts.
- `src/utils/swarm/backends/registry.ts` detects environment and returns backend implementations (`tmux`, `iterm2`, or fallback behavior), with caching.

Why it matters:
- Encapsulates terminal backend differences behind a stable interface.
- Enables platform-specific behavior without spreading conditionals everywhere.

## 3) Plugin architecture with manifest validation

Plugins follow a file-structured extension model validated by schemas:

- `src/utils/plugins/pluginLoader.ts` documents expected plugin layout and loads plugin components.
- Schema-backed validation and load/error aggregation are handled in loader flow.

Why it matters:
- Supports controlled extensibility without tightly coupling plugin logic to core runtime.
- Allows deterministic discovery precedence and policy-aware filtering.

## 4) Lightweight external store pattern

Application state is managed with a minimal custom store instead of a heavyweight framework:

- `src/state/store.ts` provides `createStore` with `getState`, `setState`, `subscribe`.
- `src/state/AppState.tsx` wraps this store in React context with selective subscriptions via `useSyncExternalStore`.

Why it matters:
- Predictable update model with low overhead.
- Works for both React-bound and non-React state consumers.

## 5) Layered context assembly

Runtime prompt context is assembled through memoized context providers:

- `src/context.ts` provides `getSystemContext()` and `getUserContext()` (git snapshot, CLAUDE.md, current date, and optional debug injection).
- `src/query.ts` prepends/appends context during request construction.

Why it matters:
- Maintains a clear boundary between conversation messages and ambient execution context.
- Reduces repeated I/O via memoization.

## 6) Feature-flag driven dead-code elimination

Build/runtime behavior is heavily feature gated:

- `feature('...')` guards appear across `entrypoints/cli.tsx`, `main.tsx`, `commands.ts`, and `tools.ts`.
- Many optional modules are loaded with conditional `require()`/dynamic import only when enabled.

Why it matters:
- Supports product variants and internal/external skew.
- Reduces startup and bundle costs by avoiding unnecessary module load.

## 7) Lazy-loading for startup performance

Startup paths avoid loading full dependency graphs until needed:

- `src/entrypoints/cli.tsx` handles fast paths and early exits before importing full CLI.
- `src/main.tsx` and `src/entrypoints/init.ts` defer expensive modules behind dynamic imports where possible.

Why it matters:
- Improves CLI responsiveness for frequent short commands and fast-path operations.

## 8) Contract-based tool execution context

Tools execute through typed contracts and shared context:

- `src/Tool.ts` defines `ToolUseContext`, permission context, and tool-related progress types.
- `src/query.ts` and tool orchestration modules consume this context to run tools consistently.

Why it matters:
- Provides a stable extension contract for built-in and dynamic tools.
- Keeps permissioning, app-state integration, and session constraints explicit.

## Related pages

- [Architecture Overview](architecture-overview.md)
- [Architecture Decisions](architecture-decisions.md)
- [Key Capabilities](key-capabilities.md)
