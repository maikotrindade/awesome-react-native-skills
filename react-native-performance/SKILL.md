---
name: react-native-performance
description: Deep React Native performance — FPS profiling, bundle size, TTI measurement, native threading, memory leaks, and profiling tools.
metadata:
  author: maikotrindade
  version: "2026.5.0"
---

# React Native Performance

> **Deep performance reference.** For high-level performance basics (FlatList tuning, memoization, Hermes intro, expo-image), see:
> - `react-native-core/best-practices-performance.md` — core RN primitives perf
> - `react-native/best-practices-performance.md` — ecosystem-level perf
> - `react-native-core/features-new-architecture.md` — Fabric/JSI/TurboModules overview
>
> This skill goes deeper: measurement methodologies, bundle treemap analysis, Hermes mmap, R8 Android, 16 KB page-size alignment, view flattening, native threading, JNI memory patterns, and the profiling toolchain.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| FPS & re-renders | FPS measurement, profile-react, React Compiler (Expo SDK 54+), atomic state, Concurrent React, Reanimated worklets (v3/v4) | [core-fps-rerenders](references/core-fps-rerenders.md) |
| Bundle size | App size and JS bundle reduction — Ruler, Emerge, source-map-explorer, Expo Atlas, tree shaking, barrel exports, R8, asset catalog, Hermes mmap, code splitting | [core-bundle-size](references/core-bundle-size.md) |

## Features

| Topic | Description | Reference |
|-------|-------------|-----------|
| Time to Interactive | TTI measurement methodology, cold-start pipeline markers, `react-native-performance` | [features-tti](references/features-tti.md) |
| Native performance | TurboModule threading, sync vs async, view flattening, native SDKs over JS polyfills, 16 KB alignment, platform setup | [features-native-performance](references/features-native-performance.md) |
| Memory | JS leak detection, native reference cycles, JNI refs, smart pointers, common leak patterns | [features-memory](references/features-memory.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| Profiling tools | DevTools flamegraph, Flashlight, Xcode Instruments, Android Studio Profiler, Perfetto, bundle analyzers | [best-practices-profiling](references/best-practices-profiling.md) |

## Key Recommendations

- **Always measure first** — never optimize without a baseline (Flashlight for FPS/TTI, source-map-explorer for bundle)
- **Hermes mmap removed in RN 0.79+** — JS bundle is now resident memory; bundle size affects startup memory directly
- **TurboModules** — prefer sync methods for fast operations; async only when work crosses threads
- **React Compiler** is opt-in via Expo SDK 54+ (`experiments.reactCompiler: true`) — replaces most `useMemo`/`useCallback`
- **Profile before refactoring** — 80% of perf wins come from removing waterfalls, expensive renders, and bundle bloat, not micro-optimizations
- **Reanimated v3 stable; v4 opt-in/beta** — pick one per project, don't mix
