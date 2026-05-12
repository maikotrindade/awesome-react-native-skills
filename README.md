A curated set of Claude skills for building production React Native apps in 2026. Each section is a self-contained skill with a `SKILL.md` entry point and topical references under `references/`.

## Skills

### [react-native-core](react-native-core/)

Native primitives and platform APIs that ship with React Native.

| Reference | Description |
|-----------|-------------|
| [core-components](react-native-core/references/core-components.md) | View, Text, Image, TextInput, ScrollView — fundamental UI building blocks |
| [core-layout](react-native-core/references/core-layout.md) | Flexbox layout system for positioning and aligning components |
| [core-styling](react-native-core/references/core-styling.md) | StyleSheet API for creating and organizing component styles |
| [features-animations](react-native-core/references/features-animations.md) | Built-in Animated API — timing, spring, composition |
| [features-easing](react-native-core/references/features-easing.md) | Easing functions for custom animation curves |
| [features-touch-handling](react-native-core/references/features-touch-handling.md) | Pressable, Touchable components, gesture responders |
| [features-keyboard](react-native-core/references/features-keyboard.md) | Keyboard API, dismiss handling, KeyboardAvoidingView |
| [components-ui](react-native-core/references/components-ui.md) | Modal, Switch, ActivityIndicator, RefreshControl |
| [features-lists](react-native-core/references/features-lists.md) | FlatList and SectionList for large scrollable lists |
| [features-colors-theming](react-native-core/references/features-colors-theming.md) | Colors, PlatformColor, Appearance API, dark mode |
| [features-platform-apis](react-native-core/references/features-platform-apis.md) | Linking, Dimensions, Platform detection, AppState |
| [features-network](react-native-core/references/features-network.md) | Fetch API for HTTP requests |
| [features-statusbar](react-native-core/references/features-statusbar.md) | Status bar appearance, style, visibility |
| [features-share-vibration](react-native-core/references/features-share-vibration.md) | Native share dialog and device vibration |
| [features-timers](react-native-core/references/features-timers.md) | setTimeout, setInterval, requestAnimationFrame |
| [features-platform-specific](react-native-core/references/features-platform-specific.md) | ToastAndroid, DrawerLayoutAndroid, ActionSheetIOS |
| [features-accessibility](react-native-core/references/features-accessibility.md) | Screen reader support, labels, roles, states |
| [features-transforms](react-native-core/references/features-transforms.md) | 2D and 3D transforms — rotate, scale, translate, skew |
| [features-new-architecture](react-native-core/references/features-new-architecture.md) | Fabric, JSI, TurboModules — default since RN 0.76 |
| [best-practices-debugging](react-native-core/references/best-practices-debugging.md) | React Native DevTools, LogBox, Dev Menu |

### [react-native-ecosystem](react-native-ecosystem/)

Production ecosystem libraries that sit on top of React Native.

| Reference | Description |
|-----------|-------------|
| [core-navigation](react-native-ecosystem/references/core-navigation.md) | React Navigation v7 — NativeStack, tabs, drawer, typed routes |
| [core-state-management](react-native-ecosystem/references/core-state-management.md) | Zustand, Jotai, Redux Toolkit — devtools and persistence |
| [core-data-fetching](react-native-ecosystem/references/core-data-fetching.md) | TanStack Query v5 + Axios — queries, mutations, offline |
| [core-typescript](react-native-ecosystem/references/core-typescript.md) | Strict config, component typing, navigation typing, zod |
| [features-reanimated](react-native-ecosystem/references/features-reanimated.md) | Reanimated v3 — shared values, worklets, layout animations |
| [features-gesture-handler](react-native-ecosystem/references/features-gesture-handler.md) | Gesture Handler v2 — Tap/Pan/Pinch, composition |
| [features-storage](react-native-ecosystem/references/features-storage.md) | MMKV, AsyncStorage, Expo SecureStore — choosing |
| [features-permissions](react-native-ecosystem/references/features-permissions.md) | react-native-permissions — check, request, rationale |
| [features-push-notifications](react-native-ecosystem/references/features-push-notifications.md) | FCM + APNs via Firebase, token registration |
| [best-practices-accessibility](react-native-ecosystem/references/best-practices-accessibility.md) | a11y props, screen reader testing, focus management |
| [advanced-deep-linking](react-native-ecosystem/references/advanced-deep-linking.md) | Universal Links, App Links, React Navigation linking |

### [react-native-expo](react-native-expo/)

Expo toolchain — Router, EAS, modules, SDK upgrades.

| Reference | Description |
|-----------|-------------|
| [core-expo-router](react-native-expo/references/core-expo-router.md) | File-based routing, layouts, route groups, API routes |
| [core-eas](react-native-expo/references/core-eas.md) | EAS Build/Update/Submit/Deploy overview, eas.json |
| [features-eas-build](react-native-expo/references/features-eas-build.md) | Build profiles, dev client, local builds, device install |
| [features-eas-update-deploy](react-native-expo/references/features-eas-update-deploy.md) | OTA updates, EAS Hosting, CI/CD YAML workflows |
| [features-expo-modules](react-native-expo/references/features-expo-modules.md) | create-expo-module, module DSL, config plugins |
| [features-upgrading](react-native-expo/references/features-upgrading.md) | SDK 53–56 breaking changes, upgrade flow, doctor |
| [best-practices-ui](react-native-expo/references/best-practices-ui.md) | expo-image, responsiveness, NativeTabs, navigation |
| [best-practices-data](react-native-expo/references/best-practices-data.md) | fetch vs React Query, expo-secure-store, env vars |

### [react-native-reusables](react-native-reusables/)

shadcn/ui-style components with NativeWind v4 and RN Primitives.

| Reference | Description |
|-----------|-------------|
| [core-overview](react-native-reusables/references/core-overview.md) | What RNR is, differences from shadcn/ui |
| [core-installation](react-native-reusables/references/core-installation.md) | CLI vs manual setup, PortalHost, dependencies |
| [core-customization](react-native-reusables/references/core-customization.md) | components.json, global.css, tailwind.config, theme.ts |
| [features-components-overview](react-native-reusables/references/features-components-overview.md) | Button, Input, Dialog, variants, asChild |
| [features-text-and-icons](react-native-reusables/references/features-text-and-icons.md) | TextClassContext inheritance, Icon wrapper for Lucide |
| [features-forms-controls](react-native-reusables/references/features-forms-controls.md) | Label, Input, Select, Checkbox, react-hook-form |
| [features-overlays-portals](react-native-reusables/references/features-overlays-portals.md) | PortalHost, Dialog/Popover, contentInsets |
| [features-registry-cli](react-native-reusables/references/features-registry-cli.md) | init, add, doctor; custom registries |
| [features-blocks](react-native-reusables/references/features-blocks.md) | Auth blocks, Clerk integration |
| [best-practices-adding-components](react-native-reusables/references/best-practices-adding-components.md) | CLI preference, path aliases, cn helper |
| [best-practices-troubleshooting](react-native-reusables/references/best-practices-troubleshooting.md) | doctor, log levels, PortalHost, aliases |

### [react-native-performance](react-native-performance/)

Deep performance work — measurement, profiling, optimization.

| Reference | Description |
|-----------|-------------|
| [core-fps-rerenders](react-native-performance/references/core-fps-rerenders.md) | FPS measurement, React Compiler, Concurrent React, worklets |
| [core-bundle-size](react-native-performance/references/core-bundle-size.md) | Ruler, Emerge, source-map-explorer, tree shaking, R8, Hermes mmap |
| [features-tti](react-native-performance/references/features-tti.md) | TTI methodology, cold-start markers, react-native-performance |
| [features-native-performance](react-native-performance/references/features-native-performance.md) | TurboModule threading, view flattening, 16 KB alignment |
| [features-memory](react-native-performance/references/features-memory.md) | JS leaks, native reference cycles, JNI refs, smart pointers |
| [best-practices-profiling](react-native-performance/references/best-practices-profiling.md) | DevTools, Flashlight, Instruments, Android Profiler, Perfetto |

### [react-native-testing](react-native-testing/)

Deep React Native Testing Library reference (v13 sync, v14 async). For end-to-end flows, use [Maestro](https://maestro.mobile.dev/).

| Reference | Description |
|-----------|-------------|
| [core-patterns](react-native-testing/references/core-patterns.md) | Query priority, render/screen, userEvent vs fireEvent, waitFor |
| [reference-v13-api](react-native-testing/references/reference-v13-api.md) | v13 API — sync render, React 18, fireEvent sync |
| [reference-v14-api](react-native-testing/references/reference-v14-api.md) | v14 API — async render, React 19, v13→v14 migration |
| [best-practices-anti-patterns](react-native-testing/references/best-practices-anti-patterns.md) | Wrong queries, waitFor misuse, missing await |

## How to use

Point Claude (or your agent harness) at this repo and load the skill whose section matches your task. The `SKILL.md` at the root of each section is the entry point; it cross-links to deeper references and to sibling skills.

## License

[GNU v3.0](LICENSE)
