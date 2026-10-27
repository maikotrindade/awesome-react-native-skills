---
name: react-native-expo-upgrading
description: Expo SDK upgrade workflow — upgrade commands, cache clearing, CNG vs bare, and SDK 53–58 breaking changes table.
---

# Upgrading Expo SDK

## Core Upgrade Flow

Upgrade one SDK at a time (or skip straight to the target when the changelog says it's safe), then let Expo align every dependency:

```bash
# 1. Upgrade expo to the target SDK and align all Expo-managed packages
npx expo install expo@^58.0.0 --fix

# 2. Check for remaining issues
npx expo-doctor@latest
```

After upgrading, recreate development builds — a dev client built for the old SDK cannot load the new one. Expo Go in the stores supports only the newest SDK; use a development build if you need to stay on an older one.

**Clear caches** after upgrading (especially after major versions):

```bash
rm -rf node_modules
rm -rf .expo
watchman watch-del-all
npm install               # or yarn / pnpm install
npx expo start --clear
```

## CNG vs Bare Workflow

**CNG (Continuous Native Generation)** — no committed `ios/` or `android/` directories:

```bash
# Regenerate native projects after upgrading (SDK 57+ cleans by default; --clean is needed on older SDKs)
npx expo prebuild --clean
```

From SDK 57, `expo prebuild` deletes and regenerates `android/` and `ios/` by default. Pass `--no-clean` to apply changes on top of existing folders.

**Bare workflow** — committed native directories: do **not** run `prebuild` (it would overwrite your native code). Apply the native diffs from the Expo native project upgrade helper by hand, then:

```bash
npx pod-install
```

After `prebuild --clean`, rebuild your dev client:

```bash
eas build -p ios --profile development
eas build -p android --profile development
```

## SDK 53–58 Breaking Changes

| SDK | Breaking change | Action required |
|-----|----------------|----------------|
| **53** | New Architecture enabled by default | Verify native module compatibility; check `global._IS_FABRIC !== undefined` |
| **53** | `resolver.unstable_enablePackageExports` on by default | Remove from `metro.config.js` if you set it explicitly |
| **54** | React 19 | Update component patterns for React 19 (no more `forwardRef` wrapper needed for ref forwarding) |
| **54** | React Compiler opt-in | Add `"experiments": { "reactCompiler": true }` to `app.json` to enable |
| **54** | Reanimated requires extra dep in New Arch | `npx expo install react-native-reanimated` + follow Reanimated upgrade guide |
| **55** | `expo-av` deprecated → `expo-audio` + `expo-video` | See migration section below |
| **55** | NativeTabs replaces JS tab implementations | Optional migration; JS tabs still work |
| **56** | `@react-navigation/*` → `expo-router/*` | Run codemod: `npx expo-codemod sdk-56-expo-router-react-navigation-replace src` |
| **56–57** | Hermes V1 memory regression with Reanimated/worklets | Use `expo@57.0.17` or later; skip SDK 56 when coming from 55 |
| **57** | React Native 0.86 (no RN breaking changes) | Upgrade normally |
| **57** | `expo prebuild` cleans native dirs by default | Use `--no-clean` to keep existing folders |
| **58** | React Native 0.88, strict TypeScript API default | Remove deep `react-native/Libraries/*` imports; type refs with `ViewInstance`, `TextInputInstance`, … |
| **58** | Removed `InteractionManager` | Use `requestIdleCallback` |
| **58** | Removed `Modal` `animated` prop and `StatusBar` `backgroundColor` / `translucent` / `networkActivityIndicatorVisible` | Use `animationType` on `Modal`; handle edge-to-edge insets instead of status bar color |
| **58** | `ImageBackground` deprecated | `View` + absolutely positioned `Image` |
| **58** | iOS UIKit scene lifecycle (`SceneDelegate.swift`) | CNG handles it; bare/custom `AppDelegate` apps follow the scene migration guide; remove `ios.enableSceneSupport` set on SDK 57 |
| **58** | iPhone apps resizable on iOS 27 | `requireFullScreen` no longer opts out; `ScreenOrientation.lockAsync` may be ignored while resizable |
| **58** | Expo Router: `expo-router/unstable-native-tabs` → `expo-router/native-tabs` | Update imports |
| **58** | Expo Router: `web.output` defaults to `"server"`; async routes on by default on web | Set `web.output: "static"` (+ `apiRoutes`) or `asyncRoutes: { web: false }` to keep SDK 57 behavior |
| **58** | Expo Router custom navigators: `withLayoutContext` replaced | Use `createStandardRouterNavigator` or `integrateWithRouter` |
| **58** | Android release builds minify with R8 by default | Add ProGuard keep rules for reflection-based libraries, or set `android.enableMinifyInReleaseBuilds: false` in `expo-build-properties` |
| **58** | Fingerprint default preset `balanced` | `app.json` version bumps and `node_modules` edits no longer change the fingerprint |
| **58** | EAS Build `sdk-58` / `latest` iOS image uses Xcode 27.1 | Remove pinned images in `eas.json` |

## Module Migrations

**expo-av → expo-audio + expo-video (SDK 55+):**

```bash
npx expo install expo-audio expo-video
npm uninstall expo-av
```

Old playback:
```ts
import { Audio } from 'expo-av';
const { sound } = await Audio.Sound.createAsync(source);
await sound.playAsync();
```

New playback:
```ts
import { useAudioPlayer } from 'expo-audio';
const player = useAudioPlayer(source);
player.play();  // hooks handle resource cleanup on unmount
```

Key differences:
- Time values changed from **milliseconds → seconds**
- Player stays paused at end of track (call `player.seekTo(0)` before replaying)
- Volume is a direct property: `player.volume = 0.5` (no more `setVolumeAsync()`)

**expo-permissions → individual permission APIs (SDK 44+, long deprecated):**

```bash
# Remove
npm uninstall expo-permissions

# Use platform APIs directly
import * as ImagePicker from 'expo-image-picker';
import * as Notifications from 'expo-notifications';
# Or react-native-permissions for cross-platform
```

**@expo/vector-icons → expo-symbols (SDK 55+):**

```bash
npx expo install expo-symbols
```

```tsx
// Old
import { Ionicons } from '@expo/vector-icons';
<Ionicons name="star" size={24} color="black" />

// New
import { SymbolView } from 'expo-symbols';
<SymbolView name="star.fill" tintColor="black" size={24} />
```

Note: `expo-symbols` uses SF Symbols on iOS and Material Symbols on Android — system fonts, no icon bundle.

## React Compiler (SDK 54+, opt-in)

The React Compiler automatically memoizes components (replaces most `useMemo`/`useCallback`):

```json
// app.json
{
  "expo": {
    "experiments": {
      "reactCompiler": true
    }
  }
}
```

After enabling, audit your codebase for mutations of state/props that would violate React's rules — the compiler enforces them strictly.

## Maintenance Tasks Post-Upgrade

- Review `expo.install.exclude` in `package.json` — remove entries that no longer need pinning
- Delete obsolete `babel.config.js` / `metro.config.js` settings that SDK now handles
- Remove unnecessary patches (`.patch` files via `patch-package`) — often fixed in new SDK
- Run `npx expo-doctor` — it flags known version incompatibilities

## Key Points

- Always: `npx expo install expo@^<SDK>.0.0 --fix` → `npx expo-doctor@latest` → clear caches → rebuild dev clients
- CNG projects: regenerate native folders after any native-affecting upgrade; bare projects apply the upgrade helper diff instead
- SDK 53: New Architecture is the default — test all native modules
- SDK 56: run the codemod before manually migrating react-navigation imports
- SDK 58: run `npx tsc --noEmit` early — the strict React Native TypeScript API surfaces most breakages as type errors
- SDK 58: test an Android **release** build before shipping — R8 is now on by default
