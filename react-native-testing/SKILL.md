---
name: react-native-testing
description: Deep React Native Testing Library (RNTL) reference — v13 (React 18 sync) and v14 (React 19 async), queries, userEvent, matchers, anti-patterns.
metadata:
  author: maikotrindade
  version: "2026.5.0"
---

# React Native Testing Library

> Deep API reference for `@testing-library/react-native` v13 and v14.
>
> **Scope:** unit + component tests with Jest + RNTL. For end-to-end UI flows on real devices/simulators, use **[Maestro](https://maestro.mobile.dev/)** — declarative YAML flows, no code, runs against the built app. RNTL and Maestro are complementary, not alternatives.

**IMPORTANT:** API signatures, sync/async behavior, and available functions differ between v13 and v14. Always check the project's `package.json` and load the matching version reference rather than relying on memorized patterns.

## Version Detection

Check `@testing-library/react-native` in `package.json`:

- **v14.x** → load [reference-v14-api](references/reference-v14-api.md) (React 19+, async APIs, `test-renderer`)
- **v13.x** → load [reference-v13-api](references/reference-v13-api.md) (React 18+, sync APIs, `react-test-renderer`)

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Core patterns | Query priority, render/screen, userEvent vs fireEvent, waitFor, 11 core rules | [core-patterns](references/core-patterns.md) |
| v13 API (React 18) | Sync render, full method tables, fireEvent sync, React 19 compat shims | [reference-v13-api](references/reference-v13-api.md) |
| v14 API (React 19) | Async render, await-based APIs, removed APIs, v13→v14 migration codemods | [reference-v14-api](references/reference-v14-api.md) |
| Anti-patterns | Wrong queries, waitFor misuse, unnecessary act, destructuring, missing await | [best-practices-anti-patterns](references/best-practices-anti-patterns.md) |

## Key Recommendations

- **Always use `screen`** for queries — never destructure from `render()`
- **`getByRole` first** with `{ name: '...' }` option (most accessibility-aware)
- **`findBy*` for async elements** — not `waitFor` + `getBy*`
- **`userEvent` over `fireEvent`** — more realistic interaction simulation
- **`queryBy*` only for non-existence assertions** (`.not.toBeOnTheScreen()`)
- **v14: always `await`** every API (including `render`, `fireEvent`, `act`)
