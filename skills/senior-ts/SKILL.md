---
name: senior-ts
description: Write JavaScript/TypeScript like a senior library author. Small pure functions, explicit return types, strict lint rules, tiny public API, no classes, no needless abstraction, comments that explain why. Use whenever writing, refactoring or reviewing JS/TS (libraries, utils, hooks, compilers, parsers), or when the user asks for clean, maintainable, readable, "senior engineer" or "expert" code.
---

# senior-ts

Code reads like one careful person wrote it. Boring, flat, typed, small.

## 1. Shape of the code

- **One job per file, one job per function.** `atom.ts` is 15 lines. `chars.ts` is constants plus one predicate. File name = what it does (`media-query.ts`, `online-status.ts`, `get-identifier.ts`), kebab-case.
- **Plain functions, not classes.** State lives in an interface + `createX()` factory (`createSource`, `createRegistry`). Data shapes are `interface`s; behavior is free functions that take the data (`point(source, offset)`).
- **Declare with `function`**, named, hoisted, easy to find in stack traces. Arrow functions only for callbacks and returned closures.
- **Early return, no `else` after return, no nested ternary, max nesting depth 4.** Flatten with guard clauses:
  ```ts
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
- **Always braces** on `if`/`for`. Never single-line `if (x) return`.
- **Module-level constants in UPPER_SNAKE** (`MEDIA`, `TAB`, `LF`, `ERROR_CONSTRUCTOR`). Name magic numbers once, comment the meaning: `export const NUMBER_SIGN = 35; // #`.
- **Const enums / tag objects over strings** for hot or serialized values; string unions for public API.
- **Cache with a module `Map`** when a lookup is repeated and the key set is small; say so in a comment (`// Lives whole process: one entry per distinct option set, stays small.`).
- **Data-driven over branching.** Rules are arrays of `{ priority, trigger, run }` objects selected and sorted once (`createRegistry`), not a growing `if/else` chain. Extension points are the same shape as built-ins.

## 2. Types

- **Explicit return types on every exported function** (`explicit-module-boundary-types`). Parameters typed, locals inferred. No `: number` on `const x = 1`.
- **No `any`.** Use `unknown` then narrow. Generics only when the caller's type flows through (`atom<T>`, `select<T extends Ranked>`).
- **`import type` / `export type` for types**; separate type imports are allowed, no duplicate value imports.
- **Derive types, do not duplicate them:** `type Point = Position['start']`.
- **`interface` for object shapes, `type` for unions/aliases.** `T[]` array syntax.
- **Overloads or call signatures in an interface** for dual-mode APIs (`interface Atom<T> { (): T; (value: T): T }`).
- **Errors:** helper that builds the error with context, caller throws. Message format `Unexpected 'X' (Expected: Y)`. Rethrow with `cause` (`preserve-caught-error`). Empty `catch` only when deliberate and commented.

## 3. API surface

- **Tiny public API.** Export only what users call. Internals stay unexported. Prefer one default or a few named exports per entry.
- **Subpath entries over a barrel when features are independent** (`my-lib/atom`, `my-lib/math`), each its own file listed in the build config. Users pay only for what they import.
- **`sideEffects: false`**, ESM first, CJS kept only if existing users need it, `types` for both. `files: ["dist", "src"]`.
- **SSR/env split at module level**, not inside hot paths: `export const useX = isServer ? serverImpl : clientImpl;`
- **Cleanup is part of the function** (`onCleanup(() => removeEventListener(...))`), never left to the caller.

## 4. Comments and docs

- Comment **why / invariant / spec reference / known cost**, never what the line does.
- `/** ... */` on interface fields that carry a non-obvious contract (`lastLine`: "speeds up lookups that move forward").
- A lint suppression must have a reason: `// oxlint-disable-next-line rule -- why`.
- Deliberate trade-offs are written down in place (`// CJS is kept so require() users on 0.x keep working.`).
- README: what it is in one line, install, minimal example, API list. Nothing more.

## 5. Performance, only where measured

- Hot paths use `charCodeAt` + numeric constants, index loops, `lastLine` cursors, binary search, and null-prototype records. Everything else stays readable.
- Keep a `benchmark/` and size test (`bundle-size.test.ts`) next to the code before optimizing; do not guess.

## 6. Tests

- Test file per module/behavior in `test/` (`blocks-leaf.test.ts`, `ext-api.test.ts`). Snapshot only for compiler/codegen output.
- Add property-style checks against a reference implementation when one exists (`equivalence.test.ts`, `corpus.test.ts`). Bundle size is a test.
- E2E for UI libs with real browser specs per component (`e2e/specs/*.spec.ts`).

## 7. Repo and tooling defaults

- pnpm workspace, `packages/*`, `type: "module"`, `changesets` for versioning.
- Build: `tsdown` (or `tsup`), `platform: 'neutral'` for libs, `dts: true`, `sourcemap: true`.
- Lint: `oxlint` (type-aware) + `oxfmt` with `singleQuote: true`. Formatter owns style; do not hand-argue formatting.
- Lint rules that shape the code (enforce when writing):
  `curly: all`, `eqeqeq: smart`, `no-else-return`, `no-nested-ternary`, `no-negated-condition`, `no-shadow`, `no-var`, `prefer-const`, `prefer-arrow-callback`, `func-names: always`, `no-use-before-define` (functions exempt), `max-depth 4`, `no-console: warn`, `no-explicit-any: warn`, `consistent-type-imports`, `explicit-function-return-type`, `sort-imports` (members sorted), `arrow-body-style: as-needed`.

## 8. Self-check before finishing

1. Can any function lose a parameter, branch, or option nobody uses? Delete it.
2. Does every exported function have a return type?
3. Any `else` after `return`, nested ternary, `any`, bare `if` without braces?
4. Does each comment say why, not what?
5. Is there one test that fails if the logic breaks?
6. Did I add a dependency a few lines could replace? Remove it.

Pairs with `ponytail` (how much to build). This skill sets how it looks.
