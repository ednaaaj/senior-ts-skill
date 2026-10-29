# senior-ts

An agent skill that makes AI coding agents write JavaScript and TypeScript the way a senior library author does: small, flat, typed, boring, and easy to maintain.

Its rules come from studying how well-maintained TypeScript libraries are structured, linted, typed, tested and published, then turning that into concrete, checkable rules an agent can follow.

## Install

```bash
npx skills add ednaaaj/senior-ts-skill
```

Works with Claude Code, Cursor, Codex, OpenCode and other agents supported by the [skills CLI](https://skills.sh). Manual install for Claude Code:

```bash
git clone https://github.com/ednaaaj/senior-ts-skill.git
cp -r senior-ts-skill/skills/senior-ts ~/.claude/skills/
```

## Use

The agent loads it automatically when it writes, refactors or reviews JS/TS. You can also ask directly:

> Use senior-ts to write a debounce util.
> Refactor this file with senior-ts.
> Review this PR against senior-ts.

## What it teaches

### 1. Shape of the code
- One job per file and per function; kebab-case file names that say what the file does.
- Plain named `function` declarations and `createX()` factories instead of classes. Data lives in an `interface`, behavior in free functions that take it.
- Early returns, no `else` after `return`, no nested ternaries, max nesting depth 4, braces on every `if` and `for`.
- Named constants for magic numbers (`export const NUMBER_SIGN = 35; // #`).
- Rules as data: sorted arrays of `{ priority, trigger, run }` objects instead of growing `if/else` chains.
- Small module-level `Map` caches, with a comment saying why they stay small.

### 2. Types
- Explicit return types on every exported function. Locals inferred.
- No `any`: `unknown` plus narrowing. Generics only when the caller's type flows through.
- `import type` / `export type`, derived types instead of duplicated ones (`type Point = Position['start']`).
- Call-signature interfaces for dual-mode APIs (`interface Atom<T> { (): T; (value: T): T }`).
- Error helpers that build errors with context; rethrow with `cause`.

### 3. Public API
- Tiny surface: export only what users call.
- Subpath entries over a barrel when features are independent (`my-lib/atom`).
- `sideEffects: false`, ESM first, CJS only when existing users need it.
- Server/client split at module level; cleanup lives inside the function.

### 4. Comments
- Explain why, invariants, spec references and known costs. Never narrate the line.
- Every lint suppression carries a reason.

### 5. Performance
- Optimize only where measured: `charCodeAt`, index loops, cursors, binary search.
- Keep a benchmark and a bundle-size test next to the code.

### 6. Tests
- One test file per behavior, snapshots only for codegen, reference-equivalence checks when a reference exists, real-browser e2e for UI libraries.

### 7. Tooling defaults
- pnpm workspace, `type: "module"`, changesets.
- tsdown/tsup with `platform: 'neutral'`, `dts: true`, `sourcemap: true`.
- oxlint (type-aware) + oxfmt with `singleQuote: true`.
- The lint rules that shape the code are listed in the skill (`curly`, `eqeqeq`, `no-else-return`, `no-nested-ternary`, `no-shadow`, `consistent-type-imports`, `explicit-function-return-type`, ...).

### 8. Self-check before finishing
A six-question checklist: remove unused options, return types present, no banned patterns, comments say why, one test fails if logic breaks, no avoidable dependency.

## Example

Before (typical agent output):

```ts
export class MediaQueryManager {
  private cache: any = {};
  getMatcher(query) {
    if (!this.cache[query]) this.cache[query] = window.matchMedia(query);
    return this.cache[query];
  }
}
```

After (senior-ts):

```ts
const MEDIA = new Map<string, MediaQueryList>();

function getMediaMatcher(query: string): MediaQueryList {
  const media = MEDIA.get(query);
  if (media) {
    return media;
  }
  const newMedia = window.matchMedia(query);
  MEDIA.set(query, newMedia);
  return newMedia;
}
```

## Pairs well with

- `ponytail` (minimal-code skill): decides how much to build. senior-ts decides how it looks.

## Scope and limits

- Covers general JS/TS library and utility style. Solid-specific patterns (signals, effects) are only touched on.
- Based on reading part of each repository, not an exhaustive audit.

## Contributing

Issues and PRs welcome. Keep `skills/senior-ts/SKILL.md` terse: every rule should be backed by something observable in real code.

## License

MIT
