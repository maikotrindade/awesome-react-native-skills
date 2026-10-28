---
name: react-native-e2e-testing
description: End-to-end testing for React Native and Expo with Maestro — YAML flows, testID selectors, subflows and scripting, permissions and deep links, E2E builds, CI on EAS Workflows and GitHub Actions, and flakiness control. Detox notes for gray-box needs.
metadata:
  author: maikotrindade
  version: "2026.10.0"
---

# React Native E2E Testing

> Targets Maestro CLI 2.x against React Native 0.8x and Expo SDK 54–58 builds. For component and unit tests with Jest + React Native Testing Library, use the `react-native-testing` skill — the two are complementary.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Maestro basics | Install, first flow, selectors and `testID`, core commands, running locally, Maestro Studio | [core-maestro](references/core-maestro.md) |

## Features

| Topic | Description | Reference |
|-------|-------------|-----------|
| Advanced flows | Subflows, env/params, JavaScript, conditions, permissions, deep links, sign-in setup, locales | [features-advanced-flows](references/features-advanced-flows.md) |
| CI | E2E build profiles, EAS Workflows `type: maestro`, GitHub Actions on emulators/simulators, reports and artifacts | [features-ci](references/features-ci.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| Stable suites | Flakiness causes, waits, test data, what to cover, suite layout, Maestro vs Detox | [best-practices-stability](references/best-practices-stability.md) |

## Key Recommendations

- **Test release-like builds** — run flows against a built app (APK / simulator `.app`), not a dev client with Metro
- **Select by `testID`** — map to `id:` in Maestro; fall back to visible text for user-facing assertions
- **Let Maestro wait** — it retries assertions automatically; use `extendedWaitUntil` instead of fixed `sleep`s
- **Reset state per flow** — `launchApp: { clearState: true }` and seeded backend data keep flows independent
- **Cover critical journeys, not every screen** — sign-in, onboarding, checkout, core create/edit flows; leave branches to component tests
- **Tag and shard in CI** — `smoke` on every PR, full suite nightly; EAS Workflows `type: maestro` or emulator runners on GitHub Actions
