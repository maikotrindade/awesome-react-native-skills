---
name: react-native-e2e-stability
description: Keeping React Native E2E suites stable and useful — flakiness causes and fixes, waiting strategy, test data and backend control, what to cover, E2E-only build switches, and when to choose Detox over Maestro.
---

# Stable E2E Suites

## Flakiness: causes and fixes

| Symptom | Cause | Fix |
|---------|-------|-----|
| Element "not found" intermittently | Network-dependent content, slow cold start | `extendedWaitUntil` on a screen marker; seed data; stub slow third-party calls |
| Tap lands on the wrong element | Animation in progress, layout shift | `waitForAnimationToEnd`; disable animations in E2E builds and on the emulator |
| Text assertion fails in CI only | Device locale, font scale, or time zone differs | Pin locale/time zone in the CI image; assert by `id` for non-copy checks |
| Flow passes alone, fails in suite | Leaked state from a previous flow | `launchApp: { clearState: true }`; unique test users per flow |
| Keyboard covers the button | Soft keyboard on small devices | `hideKeyboard` before tapping; scroll into view |
| System dialog blocks the flow | Permission, "save password", update prompts | Set `permissions` at launch; disable autofill on test devices |
| Random failures on shared staging | Other people/tests mutate data | Dedicated E2E backend or namespaced test tenants, reset per run |

Treat `retries` as a safety net, not a fix: track which flows needed a retry and fix them.

## Waiting strategy

- Assertions already retry until a timeout — don't add `sleep`-style waits before them.
- Wait for **a specific marker** (`id: "home-screen"`) after navigation, not for an arbitrary delay.
- Long operations (payment, upload): `extendedWaitUntil` with an explicit `timeout`.
- Infinite animations (spinners, Lottie, skeleton shimmer) can keep `waitForAnimationToEnd` busy — hide or freeze them when `EXPO_PUBLIC_E2E` is set.

## E2E build switches

Compile test conveniences into the E2E profile only:

```ts
export const IS_E2E = process.env.EXPO_PUBLIC_E2E === '1';

// Examples
if (IS_E2E) {
  analytics.disable();
  LogBox.ignoreAllLogs();
}
<LottieView autoPlay={!IS_E2E} ... />
```

Keep the switch list short and reviewed — every branch is code production doesn't run.

## Test data

- Create users and fixtures through a **test-only API** (`runScript` + `http`), not through the UI.
- Make data unique per run (`e2e+${timestamp}@example.com`) so parallel shards don't collide.
- Clean up in `onFlowComplete` or with a nightly purge job.
- For fully hermetic runs, point the E2E build at a mock server (MSW on a local host, WireMock) — faster and deterministic, but it won't catch contract changes with the real backend.

## What to cover

| E2E (Maestro) | Component tests (RNTL) |
|---------------|------------------------|
| Sign-up, sign-in, sign-out | Form validation messages |
| Onboarding first run | Every branch of a reducer or hook |
| Purchase / checkout / subscription | Empty, loading and error states of a list |
| Core create → edit → delete journey | Formatting, i18n strings, accessibility props |
| Deep links into key screens | Navigation param handling |
| Permission denied paths for critical features | Rendering variants |

A suite of 15–40 well-chosen flows usually gives more confidence than hundreds of shallow ones.

## Selector hygiene

- Name `testID`s by purpose and screen: `checkout-pay-button`, `product-card-${id}`.
- Don't put `testID`s on decorative views; keep them on interactive elements and screen roots.
- Lint for duplicates in lists — make IDs unique with the item key.
- Accessibility labels double as selectors and screen-reader text; keep them human-readable.

## Maestro vs Detox

| | Maestro | Detox |
|---|---|---|
| Model | Black-box, YAML flows, any build | Gray-box, JS/TS tests in Jest, instrumented build |
| Synchronization | Waits on UI stability with timeouts | Tracks RN bridge/timers/network idle internally |
| Setup cost | Low — install CLI, write YAML | Higher — native config, test runner, build variants |
| Mocking | Via backend / launch args | Jest-style module mocking in the app build |
| Best for | Product journeys, QA-authored flows, Expo apps | Teams wanting JS assertions and deep app synchronization |

Both target release-like builds. Pick one per app; running both duplicates effort.

## Key Points

- Most flakiness is state, data and animation — clear state, seed data, freeze animations in E2E builds.
- Wait on markers, not time; retries are telemetry, not a cure.
- Keep E2E for critical journeys and push branching logic down to component tests.
- Stable, purpose-named `testID`s are the cheapest stability win.
