---
name: react-native-expo-upgrading
description: Expo SDK upgrade workflow — upgrade commands, cache clearing, CNG vs bare, and SDK 53–56 breaking changes table.
---

# Upgrading Expo SDK

## Core Upgrade Flow

Run these three commands in order after every SDK version bump:

```bash
# 1. Upgrade expo and all Expo SDK packages
npx expo install expo@latest

# 2. Auto-fix dependency versions to match the new SDK
npx expo install --fix

# 3. Check for remaining issues
npx expo-doctor
```

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
# Regenerate native projects from scratch after upgrading
npx expo prebuild --clean
```

**Bare workflow** — committed native directories:

```bash
# Apply config changes after upgrade
npx expo prebuild --clean   # only when upgrading requires native changes
```

After `prebuild --clean`, rebuild your dev client:

```bash
eas build -p ios --profile development
eas build -p android --profile development
```

## SDK 53–56 Breaking Changes

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

- Always: `expo install expo@latest` → `expo install --fix` → `expo-doctor` → clear caches
- CNG projects: run `npx expo prebuild --clean` after any native-affecting upgrade
- SDK 53: New Architecture is the default — test all native modules
- SDK 56: run the codemod before manually migrating react-navigation imports
