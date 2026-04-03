# Architecture Decisions

This document records notable architecture decisions inferred from source structure and implementation comments.

## 1) Fast bootstrap with mode-specific short-circuiting

Decision:
- Handle common/special CLI modes before loading the full runtime.

Evidence:
- `src/entrypoints/cli.tsx` explicitly documents fast paths and dynamic-import strategy.

Tradeoff:
- Better startup performance and reduced module evaluation.
- More branching at bootstrap, requiring careful ordering of checks.

## 2) Centralized startup side effects

Decision:
- Put initialization side effects in a dedicated `init()` entrypoint.

Evidence:
- `src/entrypoints/init.ts` owns config enablement, safe env application, telemetry and settings/policy preloading, network setup, and cleanup registration.

Tradeoff:
- Clear startup responsibility boundary.
- Large init surface area can become a coordination hotspot.

## 3) Single source of truth for global runtime state (with explicit caution)

Decision:
- Keep shared process state in `bootstrap/state.ts`, while warning against uncontrolled growth.

Evidence:
- `src/bootstrap/state.ts` contains explicit warning comments ("DO NOT ADD MORE STATE HERE" and related cautionary notes).

Tradeoff:
- Practical central state for cross-cutting concerns.
- Risk of hidden coupling if state expands without discipline.

## 4) Registry-based capability exposure (commands/tools)

Decision:
- Use composition roots (`commands.ts`, `tools.ts`) to define model/user-visible capabilities.

Evidence:
- `src/tools.ts` marks `getAllBaseTools()` as source of truth.
- `src/commands.ts` memoizes command assembly and conditionally includes feature-gated commands.

Tradeoff:
- Easy discoverability and governance.
- Large registry files can become dense and require strong organization.

## 5) Build-time feature gating and product skew support

Decision:
- Use feature flags and env-gating to support multiple product variants from one codebase.

Evidence:
- Repeated `feature('...')` and `process.env.USER_TYPE` checks across `cli.tsx`, `main.tsx`, `commands.ts`, `tools.ts`, and `state/AppState.tsx`.

Tradeoff:
- Flexible product segmentation and dead-code elimination.
- Higher complexity in testing matrix and conditional behavior reasoning.

## 6) Plugin-first extensibility instead of hard-coding all behaviors

Decision:
- Invest in plugin loading, manifest/schema validation, and source-policy checks.

Evidence:
- `src/utils/plugins/pluginLoader.ts` handles plugin discovery precedence, validation, and load outcomes.

Tradeoff:
- Strong extensibility and customization.
- Additional safety and compatibility complexity (validation, trust, policy).

## 7) Multi-provider API abstraction

Decision:
- Keep one API-client construction path that can target Anthropic direct, Bedrock, Foundry, and Vertex.

Evidence:
- `src/services/api/client.ts` conditionally builds provider-specific clients and auth flows.

Tradeoff:
- Unified call-site behavior across providers.
- Provider-specific auth and region semantics increase maintenance burden.

## 8) Context enrichment as a first-class runtime concern

Decision:
- Build and cache system/user context (git snapshot, CLAUDE.md, date) separately from message history.

Evidence:
- `src/context.ts` memoized `getSystemContext`/`getUserContext`; `src/query.ts` applies context when building requests.

Tradeoff:
- Better contextual quality and consistency.
- More implicit prompt composition logic to reason about during debugging.

## 9) High-fan-in REPL orchestration for user-facing behavior

Decision:
- Keep `screens/REPL.tsx` as the integration point for many UX/runtime concerns.

Evidence:
- `src/screens/REPL.tsx` imports and wires command handling, tool pools, hooks, remote/session behavior, notifications, and query streaming.

Tradeoff:
- Centralized UX orchestration.
- Large file complexity; changes may have broad behavioral effects.

## 10) Defense-focused request shaping and attribution controls

Decision:
- Encode anti-distillation and request-attribution behavior directly in request construction and headers, with strong gating.

Evidence:
- `src/services/api/claude.ts` includes guarded `anti_distillation` fake-tool opt-in paths.
- `src/utils/betas.ts` includes first-party-only anti-distillation summarization beta controls.
- `src/constants/system.ts` includes gated client-attestation placeholder handling in billing headers.

Tradeoff:
- Stronger anti-copying and attribution controls where enabled.
- More complexity in request-path behavior and environment/build conditionality.

## Related pages

- [Architecture Overview](architecture-overview.md)
- [Design Patterns](design-patterns.md)
- [Runtime Flow](runtime-flow.md)
- [External Findings Cross-Check](external-findings-cross-check.md)
