---
name: react-native-ecosystem
description: React Native ecosystem — navigation, state, data fetching, Reanimated 4, worklets, Gesture Handler 3, FlashList v2, storage, TypeScript patterns, accessibility, and deep linking. Use when building production React Native apps with React Navigation, Zustand, TanStack Query, Reanimated, Gesture Handler, or FlashList.
metadata:
  author: maikotrindade
  version: "2026.10.0"
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
| Reanimated 4 | CSS animations and transitions, shared values, springs, layout animations, shared element transitions, v3 → v4 migration | [features-reanimated](references/features-reanimated.md) |
| Worklets | `react-native-worklets` runtimes, `scheduleOnUI`/`scheduleOnRN`, background worker runtimes, Synchronizables | [features-worklets](references/features-worklets.md) |
| Gesture Handler 3 | Hook API (`usePanGesture`, `useTapGesture`, …), composition hooks, cross-view relations, ReanimatedSwipeable, v2 → v3 migration | [features-gesture-handler](references/features-gesture-handler.md) |

### Lists

| Topic | Description | Reference |
|-------|-------------|-----------|
| FlashList v2 | Recycling rules, `useRecyclingState`, item types, masonry, chat lists with `maintainVisibleContentPosition`, v1 → v2 migration | [features-flashlist](references/features-flashlist.md) |

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
- **Animations**: Prefer Reanimated 4 over the legacy `Animated` API; use CSS transitions/animations for state-driven motion and shared values for gesture/scroll-driven motion
- **Threads**: Import `scheduleOnRN`/`scheduleOnUI` from `react-native-worklets`; `runOnJS`/`runOnUI` are deprecated
- **Gestures**: Write new gestures with Gesture Handler 3 hooks; the v2 `Gesture.*` builders remain only for migration
- **Long lists**: FlashList v2 with `getItemType`; never keep per-item state in plain `useState` — use `useRecyclingState`
- **Storage**: MMKV for frequent read/write (sync, 10× faster); AsyncStorage for simple persistence; SecureStore for credentials
- **TypeScript**: Enable `strict: true`; type navigation with `useNavigation<NavigationProp<RootStackParamList>>()`
