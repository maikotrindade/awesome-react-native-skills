---
name: react-native-e2e-ci
description: Running Maestro E2E tests in CI — EAS build profiles for emulator/simulator builds, EAS Workflows type maestro jobs with sharding and retries, GitHub Actions on Android emulators and iOS simulators, JUnit reports and artifacts.
---

# E2E in CI

## E2E build profile (Expo / EAS)

```json
// eas.json
{
  "build": {
    "e2e": {
      "distribution": "internal",
      "android": { "buildType": "apk" },
      "ios": { "simulator": true },
      "env": { "EXPO_PUBLIC_E2E": "1" },
      "channel": "e2e"
    }
  }
}
```

- `apk` installs directly on emulators; `simulator: true` produces a `.app` for iOS simulators (no signing needed).
- `EXPO_PUBLIC_E2E` gates test-only code paths (launch-argument sign-in, disabled animations, mock analytics). Strip them from production profiles.

## EAS Workflows

EAS Workflows has a built-in Maestro job (marked alpha by Expo) that runs flows on an Android Emulator or iOS Simulator against a build from an earlier job.

```yaml
# .eas/workflows/e2e.yml
name: E2E

on:
  pull_request:
    branches: ['main']

jobs:
  build_android:
    type: build
    params:
      platform: android
      profile: e2e

  build_ios:
    type: build
    params:
      platform: ios
      profile: e2e

  maestro_android:
    needs: [build_android]
    type: maestro
    runs_on: linux-large-nested-virtualization
    params:
      build_id: ${{ needs.build_android.outputs.build_id }}
      flow_path: ./.maestro/flows
      include_tags: smoke
      shards: 2
      retries: 1

  maestro_ios:
    needs: [build_ios]
    type: maestro
    params:
      build_id: ${{ needs.build_ios.outputs.build_id }}
      flow_path: ./.maestro/flows
      include_tags: smoke
      record_screen: true
```

| Param | Notes |
|-------|-------|
| `build_id` | Required; usually `${{ needs.<job>.outputs.build_id }}` |
| `flow_path` | File, directory, or list |
| `include_tags` / `exclude_tags` | Passed to `--include-tags` / `--exclude-tags` |
| `shards` | Parallel split (experimental) |
| `retries`, `retry_failed_only` | Retry failing flows (only failed ones by default) |
| `record_screen` | Video artifact; can slow emulators — use larger runners |
| `device_identifier` | e.g. `pixel_6`, or `{ android: ..., ios: ... }` |
| `maestro_version` | Pin the CLI version for reproducibility |
| `output_format` | `junit` by default |

Hooks (`after_checkout`, `before_maestro_tests`, `after_maestro_tests`) run setup such as seeding a backend. Use a fingerprint or repack job to avoid a full native rebuild when only JS changed.

## GitHub Actions — Android emulator

```yaml
# .github/workflows/e2e-android.yml
name: E2E Android
on: pull_request

jobs:
  maestro:
    runs-on: ubuntu-latest
    timeout-minutes: 45
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: temurin, java-version: '17' }

      - name: Enable KVM
        run: |
          echo 'KERNEL=="kvm", GROUP="kvm", MODE="0666", OPTIONS+="static_node=kvm"' | sudo tee /etc/udev/rules.d/99-kvm4all.rules
          sudo udevadm control --reload-rules && sudo udevadm trigger --name-match=kvm

      - name: Build release APK
        run: npm ci && npx expo prebuild --platform android && cd android && ./gradlew assembleRelease

      - name: Install Maestro
        run: |
          curl -fsSL "https://get.maestro.mobile.dev" | bash   # official installer
          echo "$HOME/.maestro/bin" >> "$GITHUB_PATH"

      - name: Run flows on emulator
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: 34
          arch: x86_64
          profile: pixel_6
          disable-animations: true
          script: |
            adb install android/app/build/outputs/apk/release/app-release.apk
            maestro test .maestro/flows --include-tags smoke --format junit --output maestro-report.xml
        env:
          MAESTRO_PASSWORD: ${{ secrets.E2E_PASSWORD }}

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: maestro-android
          path: |
            maestro-report.xml
            ~/.maestro/tests/**
```

The emulator runner's `script` runs each line as a separate shell command; keep multi-step logic in a script file if it grows.

## GitHub Actions — iOS simulator

```yaml
  maestro-ios:
    runs-on: macos-latest
    timeout-minutes: 60
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npx expo prebuild --platform ios
      - name: Build for simulator
        run: |
          xcodebuild -workspace ios/MyApp.xcworkspace -scheme MyApp \
            -configuration Release -sdk iphonesimulator \
            -destination 'generic/platform=iOS Simulator' \
            -derivedDataPath build
      - name: Boot simulator and install
        run: |
          UDID=$(xcrun simctl list devices available | grep -m1 "iPhone 1" | grep -Eo '[0-9A-F-]{36}')
          xcrun simctl boot "$UDID"
          xcrun simctl install "$UDID" build/Build/Products/Release-iphonesimulator/MyApp.app
      - name: Install Maestro
        run: |
          curl -fsSL "https://get.maestro.mobile.dev" | bash
          echo "$HOME/.maestro/bin" >> "$GITHUB_PATH"
      - run: maestro test .maestro/flows --include-tags smoke --format junit --output maestro-report.xml
```

macOS runners are slower and costlier; run iOS on merges to `main` or nightly if PR time matters.

## Hosted device clouds

Maestro Cloud (`maestro cloud` CLI, or its GitHub Action) and other device farms run flows in parallel on managed devices — useful when emulator time on CI runners becomes the bottleneck. Upload the same E2E build artifact.

## Reports and debugging

- `--format junit --output report.xml` → feed to the CI test summary.
- Maestro stores screenshots, logs and the view hierarchy for each run under `~/.maestro/tests/<timestamp>/`; upload it on failure.
- `--debug-output <dir>` writes debug artifacts to a known path.
- Record video for flaky flows only — it costs runner time.

## Pipeline shape

| Trigger | Suite | Platforms |
|---------|-------|-----------|
| Pull request | `smoke` tag (5–10 min) | Android (cheaper), optionally iOS |
| Merge to `main` | `smoke` + `regression` | Android + iOS |
| Nightly | Everything, including slow and visual flows | Both, several OS versions |

## Key Points

- Build once with an `e2e` profile (APK / simulator `.app`), test the artifact.
- EAS Workflows: `type: maestro` with `build_id` from the build job; tag, shard and retry there.
- GitHub Actions: enable KVM and use an emulator runner for Android; boot a simulator with `simctl` on macOS for iOS.
- Always upload JUnit and `~/.maestro/tests` artifacts so failures are debuggable.
