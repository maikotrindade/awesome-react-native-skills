---
name: react-native-expo
description: Expo toolchain for React Native — Expo Router, EAS Build/Update/Deploy, native modules, SDK upgrading, and Expo UI/data patterns.
metadata:
  author: maikotrindade
  version: "2026.5.0"
---

# React Native Expo

> Targets Expo SDK 52–56, Expo Router v4, and EAS. For ecosystem libraries (state, Reanimated, React Navigation) see the `react-native` skill. For shadcn/ui-style components see the `react-native-reusables` skill.

Expo provides a full-stack toolchain on top of React Native: file-based routing via Expo Router, cloud build/deploy infrastructure via EAS, a native module authoring API, and a curated SDK of first-party libraries. This skill covers the Expo layer exclusively.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Expo Router | File-based routing, layouts, route groups, API routes, SDK 56 migration | [core-expo-router](references/core-expo-router.md) |
| EAS Overview | EAS Build/Update/Submit/Deploy, eas.json structure, version management | [core-eas](references/core-eas.md) |

## Features

| Topic | Description | Reference |
|-------|-------------|-----------|
| EAS Build & Dev Client | Build profiles, dev client setup, local builds, device installation | [features-eas-build](references/features-eas-build.md) |
| EAS Update & Deploy | OTA updates, EAS Hosting, CI/CD YAML workflows | [features-eas-update-deploy](references/features-eas-update-deploy.md) |
| Expo Modules | create-expo-module, module DSL, config plugins | [features-expo-modules](references/features-expo-modules.md) |
| Upgrading Expo SDK | Upgrade flow, cache clearing, SDK 53–56 breaking changes, module migrations | [features-upgrading](references/features-upgrading.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| Building UI | Code style, expo-image, responsiveness, NativeTabs, navigation patterns | [best-practices-ui](references/best-practices-ui.md) |
| Data Fetching | fetch vs React Query, expo-secure-store, offline, env vars | [best-practices-data](references/best-practices-data.md) |

## Key Recommendations

- **Expo Go first** — use `npx expo start` before building a dev client; custom builds are only needed for custom native code
- **EAS manages versions** — set `appVersionSource: "remote"` in `eas.json` to avoid manual version bumps
- **File-based routing** — routes live in `app/`; never co-locate components, types, or utilities in `app/`
- **API route secrets** — use `eas env:create` for secrets; client-side env vars use `EXPO_PUBLIC_` prefix (inlined at build time)
- **SDK upgrades** — always run `npx expo install --fix` then `npx expo-doctor` after upgrading; clear caches before debugging
