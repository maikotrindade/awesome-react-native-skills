---
name: react-native-native-modules-choosing
description: Choosing how to write native code for React Native — Turbo Modules, Fabric components, Expo Modules API and Nitro Modules compared, plus app-local modules versus published libraries.
---

# Choosing an Approach

## First: do you need native code?

1. Check for a maintained library (React Native Directory, Expo SDK) that already wraps the API.
2. Check whether an Expo SDK module covers it in an Expo app.
3. Only then write a module — and keep its surface small.

## The options

| Approach | What you write | Strengths | Trade-offs |
|----------|----------------|-----------|------------|
| **Turbo Module** | TS spec + Kotlin/Java + Objective-C++ (Swift via a wrapper) | Core React Native API, no extra dependency, Codegen type safety, lazy loading, sync or async methods | Most boilerplate; iOS needs Objective-C++ glue |
| **Fabric native component** | TS props spec + Android view manager + iOS `RCTViewComponentView` | Native views with typed props, events and commands; concurrent rendering | Same boilerplate cost as Turbo Modules; C++ for custom shadow nodes |
| **Expo Modules API** | Swift + Kotlin with a declarative DSL | Least boilerplate, modern languages, works in bare and Expo apps, config plugins for native setup | Depends on `expo-modules-core`; not core RN |
| **Nitro Modules** | TS interface (`HybridObject`) + Swift/Kotlin/C++ | Very fast JSI calls, typed objects with methods and properties, Swift without Objective-C | Third-party framework (`react-native-nitro-modules`), its own codegen (`nitrogen`) |

All four run on the New Architecture through JSI. Since RN 0.82 the legacy architecture can't be turned back on; old bridge-style modules (`RCT_EXPORT_MODULE` / `ReactContextBaseJavaModule` without a spec) only work through the interop layer — migrate them to a spec.

## Decision guide

| Situation | Pick |
|-----------|------|
| Expo app, or team prefers Swift/Kotlin and minimal glue | Expo Modules API |
| Library for the whole ecosystem with no extra runtime dependency | Turbo Module / Fabric component |
| Hot path called thousands of times (crypto, storage, sensors, parsing) | Nitro Module, or a C++ Turbo Module |
| Shared C++ logic across iOS and Android | C++ Turbo Module (`cpp` language) or Nitro with C++ |
| Wrapping a native UI widget (map, video, chart, camera preview) | Fabric component, Expo view, or Nitro view |
| One-off feature inside a single app | App-local module (any approach) — no publishing |

`create-react-native-library` scaffolds Turbo modules, Fabric views, Nitro modules, Nitro views and JS-only libraries. `create-expo-module` scaffolds Expo Modules.

## Local module vs library

| | App-local | Library package |
|---|---|---|
| Location | `modules/` or `native/` folder in the app, linked through the workspace | Own repo or monorepo package, published to npm |
| Setup | `npx create-react-native-library@latest --local` or `npx create-expo-module@latest --local` | `npx create-react-native-library@latest my-lib` |
| Example app | Your app | Generated `example/` app |
| Versioning | Ships with the app | Semver, peer deps on `react-native` |

Start local; extract to a library when a second app needs it.

## Sync vs async

| Return type | Runs on | Use for |
|-------------|---------|---------|
| Plain value (sync) | JS thread, blocking | Constants, in-memory reads, trivial math (< ~1 ms) |
| `Promise<T>` | Native module's queue / background thread, resolves on JS | I/O, disk, network, keychain, anything that may block |
| Callback / event emitter | Native → JS when something happens | Streams, sensors, progress, lifecycle |

Sync calls that touch disk or the network cause dropped frames and ANRs — see `react-native-performance/features-native-performance.md`.

## Key Points

- Prefer an existing library; then Expo Modules for least glue; Turbo Modules for zero-dependency libraries; Nitro for hot paths.
- Native views → Fabric component (or Expo/Nitro view).
- Start as an app-local module; publish only when reuse demands it.
- Default to async; reserve sync methods for trivially cheap work.
