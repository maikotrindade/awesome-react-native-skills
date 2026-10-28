---
name: react-native-e2e-advanced-flows
description: Advanced Maestro flows for React Native — reusable subflows, env and parameters, JavaScript with runScript/evalScript and HTTP, conditional steps, permissions, deep links, sign-in shortcuts, launch arguments and multi-locale runs.
---

# Advanced Maestro Flows

## Suite layout

```
.maestro/
├── config.yaml
├── flows/
│   ├── auth/login.yaml
│   ├── checkout/purchase.yaml
│   └── onboarding/first-run.yaml
├── subflows/
│   ├── sign-in.yaml
│   └── dismiss-onboarding.yaml
└── scripts/
    └── seed-user.js
```

Keep `subflows/` out of `config.yaml` `flows:` globs so they don't run on their own.

## Subflows with parameters

```yaml
# subflows/sign-in.yaml
appId: com.example.app
---
- tapOn:
    id: "login-email"
- inputText: ${EMAIL}
- tapOn:
    id: "login-password"
- inputText: ${PASSWORD}
- tapOn:
    id: "login-submit"
- assertVisible:
    id: "home-screen"
```

```yaml
# flows/checkout/purchase.yaml
appId: com.example.app
tags: [regression]
---
- launchApp:
    clearState: true
- runFlow:
    file: ../../subflows/sign-in.yaml
    env:
      EMAIL: buyer@example.com
      PASSWORD: ${PASSWORD}
- tapOn: "Add to cart"
```

## Env variables

- Pass with `-e KEY=value` on the CLI, or `MAESTRO_`-prefixed shell variables (`MAESTRO_PASSWORD` → `${MAESTRO_PASSWORD}`).
- Defaults in the flow header: `env: { BASE_URL: https://staging.example.com }`.
- Never commit real credentials; inject test-account secrets from CI.

## JavaScript

```yaml
- runScript:
    file: ../../scripts/seed-user.js
    env:
      API_URL: ${API_URL}
- inputText: ${output.user.email}
- evalScript: ${output.orderTotal = 3 * 9.99}
- assertTrue: ${output.orderTotal > 20}
```

```js
// scripts/seed-user.js — Maestro's JS runtime provides http and json()
const res = http.post(`${API_URL}/test/users`, {
  headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${SEED_TOKEN}` },
  body: JSON.stringify({ plan: 'pro' }),
});
output.user = json(res.body);
```

Scripts run inside Maestro (not in the app) — use them to seed data through a test-only API, generate unique values, or compute assertions.

## Conditional steps

```yaml
# Dismiss a prompt only if it appears
- runFlow:
    when:
      visible: "Rate this app"
    commands:
      - tapOn: "Not now"

# Platform-specific steps
- runFlow:
    when:
      platform: iOS
    commands:
      - tapOn: "Allow While Using App"
```

Prefer deterministic state (clear state, launch args) over conditionals; each `when` hides a source of variance.

## Permissions

```yaml
- launchApp:
    clearState: true
    permissions:
      all: deny          # test the denied path
- tapOn: "Enable location"
- assertVisible: "Location is off"

- setPermissions:
    permissions:
      camera: unset      # reset so the system prompt shows again
```

By default Maestro allows all permissions at launch, so system dialogs don't appear unless you deny or unset them.

## Deep links and universal links

```yaml
- openLink: myapp://product/42
- assertVisible:
    id: "product-42-title"

- openLink:
    link: https://app.example.com/reset?token=${RESET_TOKEN}
    autoVerify: true   # Android: verify the App Link instead of showing a chooser
```

Expo Router: every route is linkable, so deep links are the fastest way to reach a screen without tapping through navigation.

## Skipping slow setup

Logging in through the UI in every flow is slow and multiplies flakiness. Test the login UI once; elsewhere:

1. **Launch arguments** read by a test-only code path in an E2E build:
   ```yaml
   - launchApp:
       clearState: true
       arguments:
         e2eSessionToken: ${SESSION_TOKEN}
   ```
   Read them natively (iOS `ProcessInfo.arguments` / `UserDefaults`, Android intent extras) and expose to JS through a small module — compile this path only into the E2E build profile.
2. **Deep link** to a test-only route that accepts a short-lived token — guarded by a build flag, never shipped to production.
3. **Seeded backend user** from `runScript`, then the real sign-in subflow.

## Locales and appearance

Two ways to run flows in another language:

- **Device language** — set the emulator/simulator language before the run (system settings, or a pre-configured AVD / simulator in CI). Tests the real OS path, including `getLocales()`.
- **In-app override** — pass a launch argument that your E2E build reads before i18n init:
  ```yaml
  - launchApp:
      clearState: true
      arguments:
        e2eLanguage: ar
  ```

```bash
maestro test .maestro/ --include-tags i18n
```

```yaml
- setDarkMode
- assertDarkMode
- setOrientation: LANDSCAPE
```

Run a smoke subset in an RTL locale and in dark mode to catch layout regressions.

## Screenshots and visual checks

```yaml
- takeScreenshot: checkout-summary          # saves checkout-summary.png
- assertScreenshot:
    path: baselines/checkout-summary.png    # reference image from an earlier takeScreenshot
    thresholdPercentage: 98                 # default 95
```

Visual baselines are brittle across devices and OS versions — pin the device model and OS image in CI if you use them.

## Key Points

- Factor repeated steps into parameterized subflows; keep secrets in CI env vars.
- Use `runScript` with `http` to seed data and `output` to pass values between steps.
- Control permissions and state explicitly; use `when` sparingly.
- Reach deep screens with links or launch arguments instead of tapping through login every time.
