---
name: react-native-expo-eas-build
description: EAS Build profiles, dev client setup, cloud vs local builds, and device installation for testing custom native code.
---

# EAS Build & Dev Client

EAS Build compiles native iOS and Android binaries in the cloud. The **dev client** (`expo-dev-client`) is a custom development build that replaces Expo Go when your app requires custom native code.

## When to Use a Dev Client

Use Expo Go (`npx expo start`) by default. Switch to a dev client only when your app needs:
- Local Expo modules or custom native code
- Third-party packages with native code not included in Expo Go
- Custom app config (custom app icon, bundle ID, entitlements)
- Apple Targets (widgets, extensions, etc.)

If `npx expo start` works, you don't need a dev client.

## Setup

```bash
npx expo install expo-dev-client
```

Add to `eas.json` development profile:

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "autoIncrement": true
    }
  }
}
```

## Building

**Cloud build (EAS):**

```bash
# Build and distribute internally
eas build -p ios --profile development
eas build -p android --profile development

# Build + submit to TestFlight in one step (iOS only)
eas build -p ios --profile development --submit
```

**Local build** (requires Xcode / Android Studio):

```bash
eas build -p ios --profile development --local
eas build -p android --profile development --local
# Outputs: .ipa (iOS) or .apk (Android)
```

## Installing on Device / Simulator

**iOS Simulator:**
```bash
xcrun simctl install booted MyApp.app
```

**iOS Device:**
```bash
ideviceinstaller -i MyApp.ipa
# Or drag .ipa onto Apple Configurator 2
```

**Android (device or emulator):**
```bash
adb install MyApp.apk
```

**QR code install (internal distribution):** EAS generates an install URL; scan with the device camera.

## Running the Dev Client

After installing, open the app on your device. It shows a launcher screen:
1. Enter the local Metro URL (e.g. `exp+myapp://expo-development-client?url=http://192.168.1.x:8081`)
2. Or scan the QR code from `npx expo start`
3. Or connect to the last known server automatically

```bash
npx expo start --dev-client
```

## Build Profiles

Define multiple profiles for different distribution stages:

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "autoIncrement": true,
      "env": { "APP_ENV": "development" }
    },
    "preview": {
      "distribution": "internal",
      "autoIncrement": true,
      "env": { "APP_ENV": "staging" }
    },
    "production": {
      "autoIncrement": true,
      "env": { "APP_ENV": "production" }
    }
  }
}
```

## CNG vs Bare Workflow

**CNG (Continuous Native Generation):** No `ios/` or `android/` directories. Expo generates native code on demand via `prebuild`. Recommended — simpler upgrades, cleaner repo.

```bash
npx expo prebuild          # generate ios/ and android/ locally
npx expo prebuild --clean  # regenerate from scratch (use after SDK upgrade)
```

**Bare workflow:** You committed `ios/` and `android/`. Manage native code manually; run `npx expo prebuild` to sync config changes.

## Key Points

- Install `expo-dev-client` and set `developmentClient: true` in the development profile
- Use EAS cloud builds to avoid needing Xcode/Android Studio on every machine
- `--local` flag builds locally — useful for CI machines with native tooling installed
- Re-build the dev client whenever you add or remove a native module; JS-only changes only need `eas update`

<!--
Source references:
- https://github.com/expo/skills (plugins/expo/skills/expo-dev-client/SKILL.md)
- https://github.com/expo/skills (plugins/expo/skills/expo-deployment/SKILL.md)
- https://docs.expo.dev/develop/development-builds/introduction/
-->
