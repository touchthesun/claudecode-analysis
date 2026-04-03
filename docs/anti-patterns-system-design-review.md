# Anti-Patterns in `src/`: How Not to Design These Systems

This document captures critical design findings from a review of `src/`, aimed at engineers who want concrete examples of risky architecture and implementation patterns.

The goal is not style nitpicking. The focus is maintainability risk, security posture, and operational fragility.

## 1) Untrusted Instruction Injection into System Prompt (High)

### What was found

MCP server-provided instructions are inserted directly into the system prompt:

- Source: `src/constants/prompts.ts`
- Function: `getMcpInstructions(...)`
- Behavior: `client.instructions` is concatenated verbatim into a privileged prompt section.

### Why this is a bad pattern

This creates a trust-boundary violation. External or semi-trusted systems can influence core assistant behavior at the highest instruction priority.

Consequences:

- Prompt injection at the system layer
- Unpredictable tool/permission behavior drift
- Security posture dependent on every connected server's hygiene

### Better design

- Treat external instructions as untrusted data, not first-class system policy.
- Put server instructions in a constrained, typed policy channel with allowlisted fields.
- Apply sanitization and policy conflict checks before use.
- Separate "guidance" from "hard constraints" and enforce hard constraints in code.

## 2) God Loop / Control-Plane Spaghetti in `query.ts` (Medium)

### What was found

`src/query.ts` contains a massive `while (true)` loop that interleaves many concerns:

- token budgets and continuation logic
- compaction and context collapse
- streaming lifecycle
- tool execution and retries
- queue draining and attachment injection
- memory/skill prefetch
- analytics and instrumentation

### Why this is a bad pattern

This is high coupling with low locality. A change in one pathway can regress unrelated pathways because the control flow is intertwined.

Consequences:

- Hard-to-reproduce bugs
- Large regression blast radius
- Difficult onboarding and code review
- Testability suffers because seams are weak

### Better design

- Split into a deterministic state machine with explicit transition handlers.
- Isolate side effects by concern (streaming, compaction, tool orchestration, queue integration).
- Use typed transition reasons and small pure reducers where possible.
- Add per-stage contract tests to prevent cross-stage regressions.

## 3) Overgrown `sessionStorage.ts` with Byte-Level Parsing and Global Mutable State (Medium)

### What was found

`src/utils/sessionStorage.ts` combines:

- large-scale persistence orchestration
- custom JSONL scanning/parsing at byte level
- chain graph reconstruction logic
- singleton mutable process-wide state (`project`, cleanup registration)

### Why this is a bad pattern

This produces a "critical blob module": complex internals, broad responsibility, and hidden shared state.

Consequences:

- Hard to evolve safely
- Increases risk of subtle data corruption or recovery bugs
- High cognitive load for every change

### Better design

- Split storage concerns into modules: append, index/walk, recovery, metadata, lifecycle.
- Minimize mutable singleton state; inject dependencies/context explicitly.
- Keep byte-level parsers isolated and heavily fuzz-tested.
- Prefer append-only, versioned records with explicit migration paths.

## 4) Source Files Containing Inline Base64 Sourcemaps (Medium)

### What was found

Multiple files under `src/` contain trailing `sourceMappingURL=data:application/json;base64,...` payloads.

### Why this is a bad pattern

Inlining generated payloads in source files is noisy and fragile.

Consequences:

- Review diffs become unreadable
- Merge conflicts become more likely
- Risk of accidental churn in unrelated changes

### Better design

- Keep generated sourcemaps out of hand-edited source.
- Emit sourcemaps only at build output level.
- Enforce via CI/lint guard if needed.

## 5) Prompt-Only Guardrails for Critical Behavior (Low to Medium)

### What was found

There are many "DO NOT", "NEVER", and "CRITICAL CONSTRAINTS" instructions in:

- `src/constants/prompts.ts`
- `src/utils/sideQuestion.ts`
- `src/utils/messages.ts`
- `src/utils/undercover.ts`

Some are paired with hard runtime controls (good), but others are advisory-only hidden reminders.

### Why this is a bad pattern

Prompt-only controls are brittle under distribution shift and model behavior drift.

Consequences:

- Non-deterministic policy compliance
- Hidden behavior assumptions that are difficult to validate
- Safety posture that depends on wording, not enforcement

### Better design

- Keep prompt guidance, but move critical guarantees to executable policy checks.
- Enforce "must not" behavior in tool/router layers.
- Log and test policy violations as first-class errors.

## 6) "Known Hack" Side Channels in Core Paths (Low)

### What was found

There are explicit side-channel and hack comments in core runtime paths, for example:

- `src/utils/sessionStart.ts` (`pendingInitialUserMessage` side channel)
- `src/utils/processUserInput/processBashCommand.tsx` ("TODO: Clean up this hack")

### Why this is a bad pattern

Quick fixes in core pathways tend to become permanent architecture debt.

Consequences:

- Implicit contracts not represented in types
- Harder refactors because behavior is spread across hidden channels

### Better design

- Replace side channels with explicit typed return contracts.
- Track hacks with bounded remediation plans and owner/timeline.
- Add tests that lock expected behavior before refactoring.

## Practical Design Lessons

- Do not allow external systems to write directly into your highest-privilege instruction layer.
- Do not centralize an entire runtime's control plane into one mutable mega-loop.
- Do not let persistence, parsing, and lifecycle management collapse into one giant module.
- Do not rely on prompt wording for behaviors that require strict guarantees.
- Do not normalize temporary hacks in foundational code paths.

## Suggested Next Steps (Engineering)

1. Add a trust boundary for MCP-provided instructions.
2. Carve `query.ts` into explicit stage handlers and transitions.
3. Split `sessionStorage.ts` into bounded components with stronger contracts.
4. Remove inline sourcemap artifacts from source files.
5. Convert high-impact prompt guardrails into runtime-enforced policies.
