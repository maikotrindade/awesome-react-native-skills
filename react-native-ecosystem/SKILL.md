---
name: react-native-ecosystem
description: React Native ecosystem — navigation, state, data fetching, Reanimated, storage, TypeScript patterns, accessibility, and deep linking. Use when building production React Native apps with React Navigation, Zustand, TanStack Query, or Reanimated v3.
metadata:
  author: maikotrindade
  version: "2026.1.0"
---

# React Native Ecosystem

> Covers React Native ecosystem libraries and production patterns as of 2026.
>
> **See also:**
> - Core RN primitives (View, Text, StyleSheet, FlatList, New Architecture) → `react-native-core`
> - Expo SDK and EAS tooling → `react-native-expo`
> - Deep performance work (FPS, bundle, TTI, memory, native) → `react-native-performance`
> - React Native Testing Library deep dive → `react-native-testing`

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
| Accessibility | a11y props, screen reader testing, semantic roles, focus management | [best-practices-accessibility](references/best-practices-accessibility.md) |

## Advanced

| Topic | Description | Reference |
|-------|-------------|-----------|
| Deep Linking | Universal Links, App Links, React Navigation linking config, dynamic links | [advanced-deep-linking](references/advanced-deep-linking.md) |

## Key Recommendations

- **Navigation**: Use `createNativeStackNavigator` over JS stack; type all routes with `RootStackParamList`
- **State vs server**: Zustand for client/UI state; TanStack Query for server state — never mix roles
- **Animations**: Prefer Reanimated v3 over the legacy `Animated` API; worklets run on the UI thread by default
- **Storage**: MMKV for frequent read/write (sync, 10× faster); AsyncStorage for simple persistence; SecureStore for credentials
- **TypeScript**: Enable `strict: true`; type navigation with `useNavigation<NavigationProp<RootStackParamList>>()`
