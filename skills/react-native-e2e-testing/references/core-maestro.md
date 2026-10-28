---
name: react-native-e2e-maestro
description: Maestro basics for React Native — installing the CLI, building the app under test, writing YAML flows, testID selectors, core commands, running locally and with Maestro Studio.
---

# Maestro Basics

Maestro drives the built app through the platform's accessibility layer (UIAutomator / XCUITest). Flows are YAML, there is no test code to compile, and it waits for the UI to settle automatically.

## Install

```bash
# macOS / Linux (Homebrew)
brew tap mobile-dev-inc/tap
brew install mobile-dev-inc/tap/maestro

maestro --version
```

The official install script (`get.maestro.mobile.dev`) is the other supported route; review it before running. Maestro needs Java 17+, plus Android SDK tools for emulators and Xcode for iOS simulators.

## Build the app under test

Maestro runs against an installed app. Use a release-like build with JS bundled in:

```bash
# Expo, local
npx expo run:android --variant release
npx expo run:ios --configuration Release

# Expo, EAS (see features-ci.md for the e2e profile)
eas build --profile e2e --platform android

# Bare React Native
cd android && ./gradlew assembleRelease
```

A dev client also works for writing flows (Metro must be running), but CI should test what you ship.

## Add testIDs

```tsx
<TextInput testID="login-email" accessibilityLabel="Email" ... />
<Pressable testID="login-submit" onPress={submit}>
  <Text>Sign in</Text>
</Pressable>
```

`testID` maps to `resource-id` on Android and `accessibilityIdentifier` on iOS; Maestro matches it with `id:`. On Android, a `testID` on a component that RN flattens away (a plain `View` with no styling) may not reach the native tree — put it on the pressable/input itself, or set `collapsable={false}`.

## First flow

```yaml
# .maestro/login.yaml
appId: com.example.app
name: Sign in with email
tags:
  - smoke
---
- launchApp:
    clearState: true
- tapOn:
    id: "login-email"
- inputText: "e2e@example.com"
- tapOn:
    id: "login-password"
- inputText: ${PASSWORD}
- hideKeyboard
- tapOn:
    id: "login-submit"
- assertVisible: "Welcome back"
```

The block above `---` is flow config (`appId`, `name`, `tags`, `env`, `onFlowStart`, `onFlowComplete`); commands follow.

## Selectors

| Selector | Example | Use |
|----------|---------|-----|
| Text (regex) | `tapOn: "Sign in"` / `text: "Total: .*"` | User-visible assertions |
| `id` | `id: "login-submit"` | Stable targets — preferred for taps |
| `index` | `id: "product-card"`, `index: 2` | Repeated items in lists |
| Relative | `below: "Shipping"`, `rightOf: { id: "qty" }`, `childOf: { id: "row-3" }` | Disambiguate similar elements |
| State | `enabled: true`, `checked: true`, `focused: true` | Assert control state |
| `point` | `point: "50%,90%"` | Last resort (canvas, maps) |

Text matching is a full-string regex — escape special characters (`"\\$9\\.99"`) or match partially with `.*`.

## Core commands

| Command | Purpose |
|---------|---------|
| `launchApp` | Start the app; `clearState`, `clearKeychain` (iOS), `stopApp: false`, `permissions`, `arguments` |
| `tapOn` / `doubleTapOn` / `longPressOn` | Interact with an element |
| `inputText` / `eraseText` / `pasteText` | Type into the focused field |
| `assertVisible` / `assertNotVisible` | Assert presence (auto-retries until timeout) |
| `assertTrue` | Assert a JS expression (`${output.total > 0}`) |
| `scroll` / `scrollUntilVisible` / `swipe` | Move through lists and carousels |
| `back` / `hideKeyboard` / `pressKey` | Navigation and keyboard (`pressKey: Enter`) |
| `extendedWaitUntil` | Wait for `visible` / `notVisible` with a custom `timeout` |
| `waitForAnimationToEnd` | Wait until the screen stops changing |
| `openLink` | Open a deep link / universal link |
| `takeScreenshot` / `startRecording` / `stopRecording` | Artifacts for debugging |
| `runFlow` / `runScript` / `evalScript` | Composition and JavaScript (see `features-advanced-flows.md`) |
| `setPermissions` / `setLocation` / `setAirplaneMode` / `setOrientation` | Device state |
| `repeat` / `retry` | Loops and retrying flaky external steps |

```yaml
- scrollUntilVisible:
    element:
      id: "product-42"
    direction: DOWN
    timeout: 20000
- extendedWaitUntil:
    visible: "Order confirmed"
    timeout: 15000
- copyTextFrom:
    id: "order-number"
- assertTrue: ${maestro.copiedText.length > 0}
```

## Running

```bash
maestro test .maestro/login.yaml                    # one flow
maestro test .maestro/                              # every flow in the folder
maestro test .maestro/ --include-tags smoke         # filter by tag
maestro test .maestro/ -e PASSWORD=secret123        # pass env vars
maestro test .maestro/ --format junit --output report.xml
maestro test .maestro/ --device emulator-5554       # pick a device
maestro studio                                      # inspector + flow builder in the browser
maestro hierarchy                                   # dump the current view tree
```

With several devices connected, `--device` (or `--shard-split N`) controls where flows run.

## Workspace config

```yaml
# .maestro/config.yaml
flows:
  - "flows/**"
includeTags:
  - smoke
executionOrder:
  continueOnFailure: false
  flowsOrder:
    - onboarding
    - login
```

## Key Points

- Test a built, release-like app; bundle the JS.
- Put `testID` on interactive elements and select them with `id:`.
- Flows are YAML: config block, `---`, then commands; tags drive what runs where.
- Assertions retry automatically; `maestro studio` and `maestro hierarchy` find selectors fast.
