# External Findings Cross-Check

This page cross-checks notable claims from an external analysis against the `src/` tree in this repository snapshot.

Source reviewed:
- [The Claude Code Source Leak: fake tools, frustration regexes, undercover mode, and more](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/)

## Verification summary

## 1) Anti-distillation fake tool injection

Status: `Confirmed`

Evidence:
- `src/services/api/claude.ts` includes a guarded path that sets `result.anti_distillation = ['fake_tools']`.
- The gate combines compile-time/runtime checks (including first-party beta inclusion and GrowthBook flag checks).

Inline snippet:

Source anchor: `src/services/api/claude.ts`

```ts
// src/services/api/claude.ts
if (
  feature('ANTI_DISTILLATION_CC')
    ? process.env.CLAUDE_CODE_ENTRYPOINT === 'cli' &&
      shouldIncludeFirstPartyOnlyBetas() &&
      getFeatureValue_CACHED_MAY_BE_STALE(
        'tengu_anti_distill_fake_tool_injection',
        false,
      )
    : false
) {
  result.anti_distillation = ['fake_tools']
}
```

## 2) First-party-only connector-text summarization beta

Status: `Confirmed`

Evidence:
- `src/utils/betas.ts` includes comments and gate logic for connector-text summarization as anti-distillation behavior.
- `src/constants/betas.ts` contains a related beta token (`summarize-connector-text-2026-03-13` in this snapshot).
- Inclusion is controlled through first-party-only beta logic and an experimental-beta kill switch.

Inline snippets:

Source anchor: `src/constants/betas.ts`

```ts
// src/constants/betas.ts
export const SUMMARIZE_CONNECTOR_TEXT_BETA_HEADER = feature('CONNECTOR_TEXT')
  ? 'summarize-connector-text-2026-03-13'
  : ''
```

Source anchor: `src/utils/betas.ts`

```ts
// src/utils/betas.ts
export function shouldIncludeFirstPartyOnlyBetas(): boolean {
  return (
    (getAPIProvider() === 'firstParty' || getAPIProvider() === 'foundry') &&
    !isEnvTruthy(process.env.CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS)
  )
}
```

Source anchor: `src/utils/betas.ts`

```ts
// src/utils/betas.ts
if (
  SUMMARIZE_CONNECTOR_TEXT_BETA_HEADER &&
  process.env.USER_TYPE === 'ant' &&
  includeFirstPartyOnlyBetas &&
  !isEnvDefinedFalsy(process.env.USE_CONNECTOR_TEXT_SUMMARIZATION) &&
  (isEnvTruthy(process.env.USE_CONNECTOR_TEXT_SUMMARIZATION) ||
    getFeatureValue_CACHED_MAY_BE_STALE('tengu_slate_prism', false))
) {
  betaHeaders.push(SUMMARIZE_CONNECTOR_TEXT_BETA_HEADER)
}
```

## 3) Undercover mode with no force-off path

Status: `Confirmed`

Evidence:
- `src/utils/undercover.ts` explicitly documents:
  - force ON via `CLAUDE_CODE_UNDERCOVER=1`
  - no force-OFF behavior
- The same module describes codename leak prevention behavior and one-time callout logic.

Inline snippet:

Source anchor: `src/utils/undercover.ts`

```ts
// src/utils/undercover.ts
// Activation:
//   - CLAUDE_CODE_UNDERCOVER=1 — force ON
//   - There is NO force-OFF.
export function isUndercover(): boolean {
  if (process.env.USER_TYPE === 'ant') {
    if (isEnvTruthy(process.env.CLAUDE_CODE_UNDERCOVER)) return true
    return getRepoClassCached() !== 'internal'
  }
  return false
}
```

## 4) Frustration detection via regex keyword matching

Status: `Confirmed`

Evidence:
- `src/utils/userPromptKeywords.ts` contains a regex matching frustration/profanity terms.

Inline snippet:

Source anchor: `src/utils/userPromptKeywords.ts`

```ts
// src/utils/userPromptKeywords.ts
const negativePattern =
  /\b(wtf|wth|ffs|omfg|shit(ty|tiest)?|dumbass|horrible|awful|piss(ed|ing)? off|piece of (shit|crap|junk)|what the (fuck|hell)|fucking? (broken|useless|terrible|awful|horrible)|fuck you|screw (this|you)|so frustrating|this sucks|damn it)\b/
```

## 5) Native client attestation placeholder in billing header

Status: `Confirmed`

Evidence:
- `src/constants/system.ts` documents and implements a `cch=00000` placeholder when `NATIVE_CLIENT_ATTESTATION` is enabled.
- Comments state the placeholder is overwritten by Bun's HTTP stack and references server parser tolerance for extra fields.

Inline snippet:

Source anchor: `src/constants/system.ts`

```ts
// src/constants/system.ts
// When NATIVE_CLIENT_ATTESTATION is enabled, includes a `cch=00000` placeholder.
// ... Bun's native HTTP stack ... overwrites the zeros with a computed hash.
const cch = feature('NATIVE_CLIENT_ATTESTATION') ? ' cch=00000;' : ''
const header = `x-anthropic-billing-header: cc_version=${version}; cc_entrypoint=${entrypoint};${cch}${workloadPair}`
```

## 6) Auto-compact failure circuit breaker with cost commentary

Status: `Confirmed`

Evidence:
- `src/services/compact/autoCompact.ts` includes:
  - `MAX_CONSECUTIVE_AUTOCOMPACT_FAILURES = 3`
  - inline comment citing sessions with high failure counts and estimated wasted daily calls.

Inline snippet:

Source anchor: `src/services/compact/autoCompact.ts`

```ts
// src/services/compact/autoCompact.ts
// Stop trying autocompact after this many consecutive failures.
// BQ 2026-03-10: 1,279 sessions had 50+ consecutive failures (up to 3,272)
// in a single session, wasting ~250K API calls/day globally.
const MAX_CONSECUTIVE_AUTOCOMPACT_FAILURES = 3
```

## 7) KAIROS-related autonomous/background scaffolding

Status: `Confirmed (feature-gated scaffolding present)`

Evidence:
- `feature('KAIROS')` and related flags appear across many modules (`main.tsx`, `tools.ts`, `commands.ts`, `utils/*`).
- Scheduler/cron infrastructure is present (`src/utils/cronTasks.ts`, `src/utils/cronScheduler.ts`, `src/tools/ScheduleCronTool/prompt.ts`).
- Memory-distillation references appear in comments in `src/memdir/*` and settings schema text (`auto-dream` wording in `src/utils/settings/types.ts`).

Inline snippets:

Source anchor: `src/tools/ScheduleCronTool/prompt.ts`

```ts
// src/tools/ScheduleCronTool/prompt.ts
export function isKairosCronEnabled(): boolean {
  return feature('AGENT_TRIGGERS')
    ? !isEnvTruthy(process.env.CLAUDE_CODE_DISABLE_CRON) &&
        getFeatureValue_CACHED_WITH_REFRESH(
          'tengu_kairos_cron',
          true,
          KAIROS_CRON_REFRESH_MS,
        )
    : false
}
```

Source anchor: `src/memdir/memdir.ts`

```ts
// src/memdir/memdir.ts
// Assistant sessions are effectively perpetual...
// A separate nightly /dream skill distills logs into topic
// files + MEMORY.md.
```

## 8) Additional claims requiring caution

Status: `Partially confirmed / not fully validated in this pass`

Notes:
- Some claims are directionally supported but not fully audited line-by-line here (for example, exact breadth/counts of shell security checks, specific legal/operational interpretations, and exploitability assertions).
- For architecture docs, those are best represented as:
  - source-backed technical observations, and
  - explicit separation from speculation or business interpretation.

## Documentation guidance for this repo

- Keep this page as a traceable cross-check against external commentary.
- When importing externally reported findings into architecture docs, prefer:
  - `Confirmed in source` with explicit file anchors, or
  - `Inferred/partially validated` with a short caveat.
- Avoid presenting policy/legal inference as established technical fact unless directly expressed in code comments or official docs.

## Related pages

- [Architecture Decisions](architecture-decisions.md)
- [Key Capabilities](key-capabilities.md)
- [Design Patterns](design-patterns.md)
