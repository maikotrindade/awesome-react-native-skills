---
name: react-native-ecosystem
description: React Native ecosystem — navigation, state, data fetching, Reanimated, storage, TypeScript patterns, testing, and New Architecture. Use when building production React Native apps with React Navigation, Zustand, TanStack Query, Reanimated v3, or the New Architecture.
metadata:
  author: maikotrindade
  version: "2026.1.0"
---

# React Native Ecosystem

> Covers React Native ecosystem libraries and production patterns as of 2026.
> For core RN primitives (View, Text, StyleSheet, FlatList) see the `react-native` skill.
> For Expo SDK and EAS tooling see the `react-native-expo` skill.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Navigation | React Navigation v7 — NativeStack, tabs, drawer, typed routes, modals | [core-navigation](references/core-navigation.md) |
| State Management | Zustand, Jotai, Redux Toolkit — setup, devtools, persistence patterns | [core-state-management](references/core-state-management.md) |
| Data Fetching | TanStack Query v5 + Axios — queries, mutations, infinite lists, offline | [core-data-fetching](references/core-data-fetching.md) |
| TypeScript | Strict config, component typing, navigation typing, zod validation | [core-typescript](references/core-typescript.md) |

## Features

### Animations & Gestures

| Topic | Description | Reference |
|-------|-------------|-----------|
| Reanimated v3 | useSharedValue, useAnimatedStyle, withTiming/Spring, layout animations, Keyframe | [features-reanimated](references/features-reanimated.md) |
| Gesture Handler v2 | GestureDetector, Tap/Pan/Pinch/LongPress, gesture composition, Reanimated integration | [features-gesture-handler](references/features-gesture-handler.md) |

### Device & Platform

| Topic | Description | Reference |
|-------|-------------|-----------|
| Storage | MMKV, AsyncStorage, Expo SecureStore — choosing the right solution | [features-storage](references/features-storage.md) |
| Permissions | react-native-permissions — check, request, rationale, openSettings | [features-permissions](references/features-permissions.md) |
| Push Notifications | FCM + APNs via Firebase, token registration, foreground/background handling | [features-push-notifications](references/features-push-notifications.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| Performance | FlatList tuning, memoization, Hermes, New Architecture adoption, bundle size | [best-practices-performance](references/best-practices-performance.md) |
| Testing | Jest + React Native Testing Library, mocking native modules, Maestro E2E | [best-practices-testing](references/best-practices-testing.md) |
| Accessibility | a11y props, screen reader testing, semantic roles, focus management | [best-practices-accessibility](references/best-practices-accessibility.md) |

## Advanced

| Topic | Description | Reference |
|-------|-------------|-----------|
| New Architecture | Fabric, JSI, TurboModules, Codegen — default in RN 0.76+, migration guide | [advanced-new-architecture](references/advanced-new-architecture.md) |
| Deep Linking | Universal Links, App Links, React Navigation linking config, dynamic links | [advanced-deep-linking](references/advanced-deep-linking.md) |

## Key Recommendations

- **Navigation**: Use `createNativeStackNavigator` over JS stack; type all routes with `RootStackParamList`
- **State vs server**: Zustand for client/UI state; TanStack Query for server state — never mix roles
- **Animations**: Prefer Reanimated v3 over the legacy `Animated` API; worklets run on the UI thread by default
- **Storage**: MMKV for frequent read/write (sync, 10× faster); AsyncStorage for simple persistence; SecureStore for credentials
- **TypeScript**: Enable `strict: true`; type navigation with `useNavigation<NavigationProp<RootStackParamList>>()`
- **Testing**: Use `userEvent` (RNTL) over `fireEvent` for realistic interactions; query by role/text first, `testID` last
- **New Architecture**: Enabled by default in RN 0.76+; write new native modules as TurboModules with Codegen specs
