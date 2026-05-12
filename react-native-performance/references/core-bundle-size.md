---
name: react-native-performance-bundle-size
description: Measure and shrink iOS/Android app size and JS bundles — Ruler, Emerge, source-map-explorer, Expo Atlas, tree shaking, R8, asset catalog, Hermes mmap, code splitting.
---

# Bundle & App Size

How to measure what's in your bundle and binary, then shrink it. Covers JS treemap analysis, tree shaking, barrel imports, R8, Hermes mmap, asset catalogs, and (rarely) code splitting.

## Analyze the JS Bundle

**React Native CLI (source-map-explorer):**

```bash
npx react-native bundle \
  --entry-file index.js \
  --bundle-output output.js \
  --platform ios \
  --sourcemap-output output.js.map \
  --dev false --minify true && \
npx source-map-explorer output.js --no-border-checks
```

`--no-border-checks` is required because Metro's source maps are non-standard. Treemap may lose ~30% info from mapping gaps.

**Expo (Atlas — more accurate):**

```bash
EXPO_UNSTABLE_ATLAS=true npx expo export --platform ios
npx expo-atlas
```

Non-Expo projects can use `expo-atlas-without-expo`.

**Re.Pack projects:**

```bash
rspack build --analyze                            # webpack-bundle-analyzer
npx bundle-stats --html --json stats.json         # bundle-stats / statoscope
RSDOCTOR=true npx react-native start              # Rsdoctor
```

### What to look for

| Finding | Problem | Fix |
|---------|---------|-----|
| Entire library appears | Barrel export | Direct imports / tree shaking |
| Duplicate packages | Multiple versions | Dedupe in `package.json` |
| Dev deps in bundle | Bad conditional import | Audit imports |
| Large polyfills | Unnecessary on Hermes | Remove (check Hermes coverage first) |
| Moment.js + locales | Bloat | Switch to `date-fns` or `dayjs` |
| `aws-sdk` (full) | 200+ KB | Use `@aws-sdk/client-*` |

## Analyze App Binary Size

**Android (Ruler — Spotify):**

```groovy
// android/build.gradle
buildscript {
  dependencies {
    classpath("com.spotify.ruler:ruler-gradle-plugin:2.0.0-beta-3")
  }
}
```

```groovy
// android/app/build.gradle
apply plugin: "com.spotify.ruler"

ruler {
  abi.set("arm64-v8a")
  locale.set("en")
  screenDensity.set(480)
  sdkVersion.set(34)

  verification {                                    // CI threshold
    downloadSizeThreshold = 20 * 1024 * 1024
    installSizeThreshold  = 50 * 1024 * 1024
  }
}
```

```bash
cd android && ./gradlew analyzeReleaseBundle      # HTML report
```

**iOS (Xcode thinning report):**

```bash
cd ios && xcodebuild -exportArchive \
  -archivePath MyApp.xcarchive \
  -exportPath ./export \
  -exportOptionsPlist ExportOptions.plist
# → "App Thinning Size Report.txt": compressed = download, uncompressed = install
```

App Store Connect (post-TestFlight upload) gives the most accurate per-variant table. **Emerge Tools** is cross-platform with treemap X-Ray and CI API — ignore "remove Hermes" type suggestions.

| Optimization | Typical reduction |
|--------------|-------------------|
| Enable R8 (Android) | ~30% |
| Remove unused polyfills | 400+ KB |
| iOS asset catalog | 10-50% of assets |
| Tree shaking | 10-15% |

Google: every 6 MB increase → 1% drop in installs.

## Avoid Barrel Exports

```tsx
// BAD — loads the whole barrel, evaluates Card/Modal/Sidebar too
import { Button } from './components';

// GOOD
import Button from './components/Button';
```

Barrels also create circular-dependency warnings (which break HMR) and inflate TTI by evaluating unused modules.

**Enforce:**

```bash
npm install -D eslint-plugin-no-barrel-files
```

```javascript
// eslint.config.js
import noBarrelFiles from 'eslint-plugin-no-barrel-files';
export default [{
  plugins: { 'no-barrel-files': noBarrelFiles },
  rules: { 'no-barrel-files/no-barrel-files': 'error' },
}];
```

**Library-specific transforms** (e.g. React Native Paper):

```javascript
// babel.config.js
module.exports = { plugins: ['react-native-paper/babel'] };
```

Public-package API can keep a barrel; internal code uses direct imports.

## Tree Shaking

| Bundler | Tree shaking | Notes |
|---------|--------------|-------|
| Metro | No | Use `@rnx-kit/metro-serializer-esbuild` |
| Expo SDK 52+ | Experimental | Config required |
| Re.Pack | Yes | Built-in via Rspack/Webpack |

**Expo SDK 52+:**

```javascript
// metro.config.js
const { getDefaultConfig } = require('expo/metro-config');
const config = getDefaultConfig(__dirname);
config.transformer.getTransformOptions = async () => ({
  transform: { experimentalImportSupport: true },
});
module.exports = config;
```

```bash
# .env
EXPO_UNSTABLE_METRO_OPTIMIZE_GRAPH=1
EXPO_UNSTABLE_TREE_SHAKING=1
```

**Bare RN:** must also set `disableImportExportTransform: true` in Babel preset to match Metro's `experimentalImportSupport`.

```javascript
module.exports = {
  presets: [['module:@react-native/babel-preset', { disableImportExportTransform: true }]],
};
```

### Platform shaking

```tsx
import { Platform } from 'react-native';      // MUST be direct import

if (Platform.OS === 'ios') { /* stripped on Android */ }
```

Namespace import (`import * as RN from 'react-native'`) defeats it.

Measured impact on bare CLI: -5% Hermes bytecode, -15% minified JS.

### Requirements

- ESM imports only (`import`, not `require`)
- Libraries declare `"sideEffects": false` (or list side-effecting files) in `package.json`
- Only runs in production builds

Verify by searching the bundle for a function you know is unused — if found, tree shaking isn't working.

## Library Size Check

Before adding a dep: [bundlephobia.com](https://bundlephobia.com) or `npx bundle-phobia-cli <pkg>`. Fallback: [pkg-size.dev](https://pkg-size.dev). VS Code extension: "Import Cost" (Webpack-based, may miss RN-specific packages).

| Size (gzipped) | Verdict |
|----------------|---------|
| <5 KB | Fine |
| 5-20 KB | Evaluate need |
| 20-50 KB | Look for alternatives |
| >50 KB | Strong justification |

| Library | Gzipped | Alternative |
|---------|---------|-------------|
| moment | ~70 KB | dayjs (~3 KB) |
| lodash (full) | ~25 KB | `lodash-es` + direct imports |
| aws-sdk (full) | 200+ KB | `@aws-sdk/client-*` |
| crypto-js | ~15 KB | `react-native-quick-crypto` |

```tsx
import _ from 'lodash';              // 71.5 KB
import get from 'lodash/get';        // 8 KB
const v = obj?.path?.to?.value;      // 0 KB
```

## R8 (Android)

```groovy
// android/app/build.gradle
android {
  buildTypes {
    release {
      minifyEnabled true
      shrinkResources true                  // requires minifyEnabled
      proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
  }
}
```

```bash
cd android && ./gradlew assembleRelease   # always QA the release build
```

Typical: 9.5 MB → 6.3 MB (33% smaller). React Native defaults are usually enough — only add rules when a library breaks.

```proguard
# proguard-rules.pro — only as needed

# Firebase
-keep class io.invertase.firebase.** { *; }
-dontwarn io.invertase.firebase.**

# Retrofit
-keepattributes Signature
-keepattributes *Annotation*
-keep class retrofit2.** { *; }
-dontwarn retrofit2.**

# Gson
-keep class com.google.gson.** { *; }

# OkHttp
-dontwarn okhttp3.**
-dontwarn okio.**

# DoNotStrip annotation
-keep @com.facebook.proguard.annotations.DoNotStrip class *
-keepclassmembers class * { @com.facebook.proguard.annotations.DoNotStrip *; }

# Disable obfuscation for readable crashes
-dontobfuscate
```

**Debug R8 crash:** find the missing class in the stack trace, add `-keep class com.example.X { *; }`, rebuild. Code using reflection is the most common breakage. Don't over-keep — that negates the savings.

## Hermes Bundle Compression (RN ≤ 0.78)

Android compresses files in the APK/AAB by default. Compressed files can't be mmap'd, so Hermes must decompress the whole bundle before reading — slower startup.

```groovy
// android/app/build.gradle — RN 0.78 and earlier ONLY
android {
  androidResources {
    noCompress += ["bundle"]
  }
}
```

**Default in RN 0.79+ — skip there.**

Trade-off: install size +8%, TTI -16% (e.g. 75.9 MB → 82 MB but 450ms faster startup). Verify:

```bash
unzip app-release.apk -d apk-contents
file apk-contents/assets/index.android.bundle
# Should show "data", not "gzip compressed"
```

Expo: `npx expo prebuild` first, then edit `android/`, or use a config plugin.

## Native Asset Delivery

**Android: automatic.** AAB packs density-specific drawables (`drawable-mdpi-v4` ... `drawable-xxxhdpi-v4`); Play Store delivers only the right one. No config required.

**iOS: opt-in via Asset Catalog.** Create `ios/RNAssets.xcassets/` (name is hardcoded). In Xcode "Bundle React Native code and images" Build Phase, add before line 8:

```bash
export EXTRA_PACKAGER_ARGS="--asset-catalog-dest ./"
```

Then build:

```bash
npx react-native run-ios --mode Release
```

Resulting catalog (per image):

```
ios/RNAssets.xcassets/assets_image_image.imageset/
├── Contents.json
├── image.jpg
├── image@2x.jpg
└── image@3x.jpg
```

App Store then delivers only the resolution the device needs — typically 30-50% asset savings.

Use `@1x` / `@2x` / `@3x` filename suffixes; consider WebP for photos, SVG for icons, and `react-native-fast-image` for caching.

## Code Splitting (Re.Pack — rarely needed)

Skip with Hermes — its mmap is already efficient and chunking gives marginal or negative wins. Only consider when:

- Not using Hermes (JSC / V8)
- App > 200 MB (Play Store limit)
- Micro-frontend / permission-gated features
- All other optimizations exhausted

```bash
npx @callstack/repack-init
```

```jsx
// Lazy chunk
const SettingsScreen = React.lazy(() =>
  import(/* webpackChunkName: "settings" */ './screens/SettingsScreen')
);

<Suspense fallback={<Loading />}><SettingsScreen /></Suspense>
```

```jsx
// index.js
import { ScriptManager, Script } from '@callstack/repack/client';

const CHUNK_URLS = {
  settings: 'https://assets.example.com/app/v42/settings.chunk.bundle',
};

ScriptManager.shared.addResolver((scriptId) => ({
  url: __DEV__ ? Script.getDevServerURL(scriptId) : CHUNK_URLS[scriptId]
    ?? (() => { throw new Error(`Unknown chunk: ${scriptId}`); })(),
  cache: { enabled: true, path: `${FileSystem.cacheDirectory}/chunks/` },
}));
```

**Security:** chunks are executable code. Only serve from a first-party HTTPS origin you control, with a fixed allowlist or release manifest. Never load from user input or third-party domains — that's remote code execution.

## Key Points

- Measure JS bundle with `source-map-explorer` (RN CLI) or Expo Atlas (more accurate for Expo); analyze the app binary with Ruler (Android), Xcode thinning report (iOS), or Emerge Tools (both).
- Barrel imports inflate bundle and TTI by evaluating every re-exported module — refactor to direct imports and enforce with `eslint-plugin-no-barrel-files`.
- Tree shaking on Expo SDK 52+ needs `experimentalImportSupport` + `EXPO_UNSTABLE_TREE_SHAKING=1`; bare RN also needs `disableImportExportTransform: true`. Direct `import { Platform } from 'react-native'` is required for platform shaking.
- R8 typically cuts Android size ~30%. React Native defaults suffice; add `-keep` rules only when a release build crashes. Always QA the release variant.
- `noCompress += ["bundle"]` only matters on RN ≤ 0.78 — trades ~8% install size for ~16% faster TTI by enabling Hermes mmap. Default in 0.79+.
- Re.Pack code splitting is rarely worth it on Hermes. If used, treat remote chunks as RCE — first-party HTTPS, allowlist, fail-closed on unknown `scriptId`.
