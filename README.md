# awesome-react-native-skills

A curated set of Claude skills for building production React Native apps in 2026. Each section is a self-contained skill with a `SKILL.md` entry point and topical references under `references/`.

## Layout

The skills are layered, not overlapping. Each topic has exactly one canonical home:

| Section | Use when… |
|---------|-----------|
| [react-native-core](react-native-core/) | Working with native primitives — View/Text/Image, Flexbox, StyleSheet, FlatList, Animated, platform APIs, New Architecture overview. |
| [react-native-ecosystem](react-native-ecosystem/) | Wiring up ecosystem libraries — React Navigation v7, Zustand/Jotai/Redux, TanStack Query, Reanimated v3, Gesture Handler, MMKV, permissions, push, deep linking, accessibility. |
| [react-native-expo](react-native-expo/) | Using the Expo toolchain — Expo Router, EAS Build/Update/Submit, Dev Client, Expo Modules, SDK upgrades. |
| [react-native-reusables](react-native-reusables/) | Building UI with the shadcn/ui-style component library on NativeWind v4 + RN Primitives. |
| [react-native-performance](react-native-performance/) | Deep performance work — FPS profiling, re-render reduction, bundle size, TTI measurement, native threading/TurboModules, memory leaks, profiling tooling. |
| [react-native-testing](react-native-testing/) | Deep React Native Testing Library reference — v13 (React 18, sync) vs v14 (React 19, async), queries, userEvent, anti-patterns. (For E2E flows, use [Maestro](https://maestro.mobile.dev/).) |

## How to use

Point Claude (or your agent harness) at this repo and load the skill whose section matches your task. The `SKILL.md` at the root of each section is the entry point; it cross-links to deeper references and to sibling skills.

## License

[MIT](LICENSE)
