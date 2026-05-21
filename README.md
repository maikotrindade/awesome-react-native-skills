A curated set of Claude skills for building production React Native apps in 2026, distributed as a Claude Code marketplace plugin. Each section is a self-contained skill with a `SKILL.md` entry point and topical references under `references/`.

## Install

In Claude Code:

```
/plugin marketplace add maikotrindade/awesome-react-native-skills
/plugin install awesome-react-native-skills@awesome-react-native-skills
```

## Skills

### [react-native-core](skills/react-native-core/)

Native primitives and platform APIs that ship with React Native.

| Reference | Description |
|-----------|-------------|
| [core-components](skills/react-native-core/references/core-components.md) | View, Text, Image, TextInput, ScrollView — fundamental UI building blocks |
| [core-layout](skills/react-native-core/references/core-layout.md) | Flexbox layout system for positioning and aligning components |
| [core-styling](skills/react-native-core/references/core-styling.md) | StyleSheet API for creating and organizing component styles |
| [features-animations](skills/react-native-core/references/features-animations.md) | Built-in Animated API — timing, spring, composition |
| [features-easing](skills/react-native-core/references/features-easing.md) | Easing functions for custom animation curves |
| [features-touch-handling](skills/react-native-core/references/features-touch-handling.md) | Pressable, Touchable components, gesture responders |
| [features-keyboard](skills/react-native-core/references/features-keyboard.md) | Keyboard API, dismiss handling, KeyboardAvoidingView |
| [components-ui](skills/react-native-core/references/components-ui.md) | Modal, Switch, ActivityIndicator, RefreshControl |
| [features-lists](skills/react-native-core/references/features-lists.md) | FlatList and SectionList for large scrollable lists |
| [features-colors-theming](skills/react-native-core/references/features-colors-theming.md) | Colors, PlatformColor, Appearance API, dark mode |
| [features-platform-apis](skills/react-native-core/references/features-platform-apis.md) | Linking, Dimensions, Platform detection, AppState |
| [features-network](skills/react-native-core/references/features-network.md) | Fetch API for HTTP requests |
| [features-statusbar](skills/react-native-core/references/features-statusbar.md) | Status bar appearance, style, visibility |
| [features-share-vibration](skills/react-native-core/references/features-share-vibration.md) | Native share dialog and device vibration |
| [features-timers](skills/react-native-core/references/features-timers.md) | setTimeout, setInterval, requestAnimationFrame |
| [features-platform-specific](skills/react-native-core/references/features-platform-specific.md) | ToastAndroid, DrawerLayoutAndroid, ActionSheetIOS |
| [features-accessibility](skills/react-native-core/references/features-accessibility.md) | Screen reader support, labels, roles, states |
| [features-transforms](skills/react-native-core/references/features-transforms.md) | 2D and 3D transforms — rotate, scale, translate, skew |
| [features-new-architecture](skills/react-native-core/references/features-new-architecture.md) | Fabric, JSI, TurboModules — default since RN 0.76 |
| [best-practices-debugging](skills/react-native-core/references/best-practices-debugging.md) | React Native DevTools, LogBox, Dev Menu |

### [react-native-ecosystem](skills/react-native-ecosystem/)

Production ecosystem libraries that sit on top of React Native.

| Reference | Description |
|-----------|-------------|
| [core-navigation](skills/react-native-ecosystem/references/core-navigation.md) | React Navigation v7 — NativeStack, tabs, drawer, typed routes |
| [core-state-management](skills/react-native-ecosystem/references/core-state-management.md) | Zustand, Jotai, Redux Toolkit — devtools and persistence |
| [core-data-fetching](skills/react-native-ecosystem/references/core-data-fetching.md) | TanStack Query v5 + Axios — queries, mutations, offline |
| [core-typescript](skills/react-native-ecosystem/references/core-typescript.md) | Strict config, component typing, navigation typing, zod |
| [features-reanimated](skills/react-native-ecosystem/references/features-reanimated.md) | Reanimated v3 — shared values, worklets, layout animations |
| [features-gesture-handler](skills/react-native-ecosystem/references/features-gesture-handler.md) | Gesture Handler v2 — Tap/Pan/Pinch, composition |
| [features-storage](skills/react-native-ecosystem/references/features-storage.md) | MMKV, AsyncStorage, Expo SecureStore — choosing |
| [features-permissions](skills/react-native-ecosystem/references/features-permissions.md) | react-native-permissions — check, request, rationale |
| [features-push-notifications](skills/react-native-ecosystem/references/features-push-notifications.md) | FCM + APNs via Firebase, token registration |
| [best-practices-accessibility](skills/react-native-ecosystem/references/best-practices-accessibility.md) | a11y props, screen reader testing, focus management |
| [advanced-deep-linking](skills/react-native-ecosystem/references/advanced-deep-linking.md) | Universal Links, App Links, React Navigation linking |

### [react-native-expo](skills/react-native-expo/)

Expo toolchain — Router, EAS, modules, SDK upgrades.

| Reference | Description |
|-----------|-------------|
| [core-expo-router](skills/react-native-expo/references/core-expo-router.md) | File-based routing, layouts, route groups, API routes |
| [core-eas](skills/react-native-expo/references/core-eas.md) | EAS Build/Update/Submit/Deploy overview, eas.json |
| [features-eas-build](skills/react-native-expo/references/features-eas-build.md) | Build profiles, dev client, local builds, device install |
| [features-eas-update-deploy](skills/react-native-expo/references/features-eas-update-deploy.md) | OTA updates, EAS Hosting, CI/CD YAML workflows |
| [features-expo-modules](skills/react-native-expo/references/features-expo-modules.md) | create-expo-module, module DSL, config plugins |
| [features-upgrading](skills/react-native-expo/references/features-upgrading.md) | SDK 53–56 breaking changes, upgrade flow, doctor |
| [best-practices-ui](skills/react-native-expo/references/best-practices-ui.md) | expo-image, responsiveness, NativeTabs, navigation |
| [best-practices-data](skills/react-native-expo/references/best-practices-data.md) | fetch vs React Query, expo-secure-store, env vars |

### [react-native-reusables](skills/react-native-reusables/)

shadcn/ui-style components with NativeWind v4 and RN Primitives.

| Reference | Description |
|-----------|-------------|
| [core-overview](skills/react-native-reusables/references/core-overview.md) | What RNR is, differences from shadcn/ui |
| [core-installation](skills/react-native-reusables/references/core-installation.md) | CLI vs manual setup, PortalHost, dependencies |
| [core-customization](skills/react-native-reusables/references/core-customization.md) | components.json, global.css, tailwind.config, theme.ts |
| [features-components-overview](skills/react-native-reusables/references/features-components-overview.md) | Button, Input, Dialog, variants, asChild |
| [features-text-and-icons](skills/react-native-reusables/references/features-text-and-icons.md) | TextClassContext inheritance, Icon wrapper for Lucide |
| [features-forms-controls](skills/react-native-reusables/references/features-forms-controls.md) | Label, Input, Select, Checkbox, react-hook-form |
| [features-overlays-portals](skills/react-native-reusables/references/features-overlays-portals.md) | PortalHost, Dialog/Popover, contentInsets |
| [features-registry-cli](skills/react-native-reusables/references/features-registry-cli.md) | init, add, doctor; custom registries |
| [features-blocks](skills/react-native-reusables/references/features-blocks.md) | Auth blocks, Clerk integration |
| [best-practices-adding-components](skills/react-native-reusables/references/best-practices-adding-components.md) | CLI preference, path aliases, cn helper |
| [best-practices-troubleshooting](skills/react-native-reusables/references/best-practices-troubleshooting.md) | doctor, log levels, PortalHost, aliases |

### [react-native-performance](skills/react-native-performance/)

Deep performance work — measurement, profiling, optimization.

| Reference | Description |
|-----------|-------------|
| [core-fps-rerenders](skills/react-native-performance/references/core-fps-rerenders.md) | FPS measurement, React Compiler, Concurrent React, worklets |
| [core-bundle-size](skills/react-native-performance/references/core-bundle-size.md) | Ruler, Emerge, source-map-explorer, tree shaking, R8, Hermes mmap |
| [features-tti](skills/react-native-performance/references/features-tti.md) | TTI methodology, cold-start markers, react-native-performance |
| [features-native-performance](skills/react-native-performance/references/features-native-performance.md) | TurboModule threading, view flattening, 16 KB alignment |
| [features-memory](skills/react-native-performance/references/features-memory.md) | JS leaks, native reference cycles, JNI refs, smart pointers |
| [best-practices-profiling](skills/react-native-performance/references/best-practices-profiling.md) | DevTools, Flashlight, Instruments, Android Profiler, Perfetto |

### [react-native-testing](skills/react-native-testing/)

Deep React Native Testing Library reference (v13 sync, v14 async). For end-to-end flows, use [Maestro](https://maestro.mobile.dev/).

| Reference | Description |
|-----------|-------------|
| [core-patterns](skills/react-native-testing/references/core-patterns.md) | Query priority, render/screen, userEvent vs fireEvent, waitFor |
| [reference-v13-api](skills/react-native-testing/references/reference-v13-api.md) | v13 API — sync render, React 18, fireEvent sync |
| [reference-v14-api](skills/react-native-testing/references/reference-v14-api.md) | v14 API — async render, React 19, v13→v14 migration |
| [best-practices-anti-patterns](skills/react-native-testing/references/best-practices-anti-patterns.md) | Wrong queries, waitFor misuse, missing await |

### [react-native-brownfield-migration](skills/react-native-brownfield-migration/)

Incremental adoption of React Native or Expo inside existing native iOS/Android apps.

| Reference | Description |
|-----------|-------------|
| [quick-start](skills/react-native-brownfield-migration/references/quick-start.md) | Shared preflight and path-selection gate (Expo vs bare) |
| [expo-create-app](skills/react-native-brownfield-migration/references/expo-create-app.md) | Scaffold a new Expo app before brownfield setup |
| [expo-quick-start](skills/react-native-brownfield-migration/references/expo-quick-start.md) | Expo plugin setup and packaging readiness |
| [expo-ios-integration](skills/react-native-brownfield-migration/references/expo-ios-integration.md) | Expo iOS packaging and host startup integration |
| [expo-android-integration](skills/react-native-brownfield-migration/references/expo-android-integration.md) | Expo Android packaging, publish, and host integration |
| [bare-quick-start](skills/react-native-brownfield-migration/references/bare-quick-start.md) | Bare React Native baseline setup |
| [bare-ios-xcframework-generation](skills/react-native-brownfield-migration/references/bare-ios-xcframework-generation.md) | Bare iOS XCFramework generation |
| [bare-android-aar-generation](skills/react-native-brownfield-migration/references/bare-android-aar-generation.md) | Bare Android AAR generation and publish |
| [bare-ios-native-integration](skills/react-native-brownfield-migration/references/bare-ios-native-integration.md) | Bare iOS host integration |
| [bare-android-native-integration](skills/react-native-brownfield-migration/references/bare-android-native-integration.md) | Bare Android host integration |

### [upgrading-react-native](skills/upgrading-react-native/)

Full React Native upgrade workflow — rn-diff-purge diffs, dependencies, React, Expo SDK alignment.

| Reference | Description |
|-----------|-------------|
| [upgrading-react-native](skills/upgrading-react-native/references/upgrading-react-native.md) | Router — choose the right upgrade path |
| [upgrade-helper-core](skills/upgrading-react-native/references/upgrade-helper-core.md) | Core Upgrade Helper workflow and reliability gates |
| [upgrading-dependencies](skills/upgrading-react-native/references/upgrading-dependencies.md) | Dependency compatibility checks and migration planning |
| [react](skills/upgrading-react-native/references/react.md) | React and React 19 upgrade alignment rules |
| [expo-sdk-upgrade](skills/upgrading-react-native/references/expo-sdk-upgrade.md) | Expo SDK-specific upgrade layer (conditional) |
| [upgrade-verification](skills/upgrading-react-native/references/upgrade-verification.md) | Manual post-upgrade verification checklist |
| [monorepo-singlerepo-targeting](skills/upgrading-react-native/references/monorepo-singlerepo-targeting.md) | Monorepo and single-repo command scoping |

### [github](skills/github/)

GitHub workflow patterns using `gh` CLI — PRs, stacked PRs, squash merge, rebase.

| Reference | Description |
|-----------|-------------|
| [stacked-pr-workflow](skills/github/references/stacked-pr-workflow.md) | Merge stacked PRs into main as individual squash commits |

### [github-actions](skills/github-actions/)

GitHub Actions patterns for React Native iOS simulator and Android emulator cloud builds with downloadable artifacts.

| Reference | Description |
|-----------|-------------|
| [gha-ios-composite-action](skills/github-actions/references/gha-ios-composite-action.md) | Composite `action.yml` for iOS simulator `.app.tar.gz` builds |
| [gha-android-composite-action](skills/github-actions/references/gha-android-composite-action.md) | Composite `action.yml` for Android emulator `.apk` builds |
| [gha-workflow-and-downloads](skills/github-actions/references/gha-workflow-and-downloads.md) | End-to-end workflow wiring plus `gh` and REST download commands |

## How to use

Install via the marketplace commands above, or point Claude (or your agent harness) directly at this repo and load the skill whose section matches your task. The `SKILL.md` at the root of each section is the entry point; it cross-links to deeper references and to sibling skills.

## License

[GNU v3.0](LICENSE)
