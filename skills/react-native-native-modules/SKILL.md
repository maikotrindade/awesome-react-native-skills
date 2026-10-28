---
name: react-native-native-modules
description: Writing native code for React Native's New Architecture — choosing between Turbo Modules, Fabric components, Expo Modules and Nitro, Codegen specs, Kotlin and Objective-C++/Swift implementations, events, threading, and scaffolding/publishing libraries with create-react-native-library.
metadata:
  author: maikotrindade
  version: "2026.10.0"
---

# React Native Native Modules

> Targets React Native 0.8x, where the New Architecture is the only architecture. For the Expo Modules API (Swift/Kotlin DSL, config plugins) see `react-native-expo/features-expo-modules.md`. For JSI/Fabric concepts see `react-native-core/features-new-architecture.md`; for threading performance see `react-native-performance/features-native-performance.md`.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Choosing an approach | Turbo Module vs Fabric component vs Expo Module vs Nitro, local module vs library | [core-choosing](references/core-choosing.md) |
| Turbo Modules | TypeScript spec, Codegen config, type mapping, Kotlin and Objective-C++ implementations, Swift, promises, events, app-local modules | [core-turbo-modules](references/core-turbo-modules.md) |

## Features

| Topic | Description | Reference |
|-------|-------------|-----------|
| Fabric components | `codegenNativeComponent`, props, direct events, commands, Android view manager, iOS component view | [features-fabric-components](references/features-fabric-components.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| Library authoring | create-react-native-library, builder-bob, example app, threading, errors, testing, Expo config plugin, publishing | [best-practices-library-authoring](references/best-practices-library-authoring.md) |

## Key Recommendations

- **Use an existing library or Expo Module first** — write a Turbo Module when you need direct control or no Expo dependency
- **The TypeScript spec is the contract** — Codegen generates the native interfaces from `Native*.ts` / `*NativeComponent.ts`; never hand-edit generated code
- **Sync only for cheap work** — sync methods block the JS thread; return a `Promise` for I/O, disk, network, or anything over a few milliseconds
- **Scaffold with `create-react-native-library`** — it wires Codegen, podspec, Gradle, an example app and builder-bob correctly; use `--local` inside an app
- **Keep native threading explicit** — UI work on the main thread, everything else on a background queue/executor
- **Test on both platforms in release builds** — Codegen and linking errors often only show up in clean native builds
