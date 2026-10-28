---
name: react-native-native-modules-library-authoring
description: Authoring native React Native libraries — scaffolding with create-react-native-library, builder-bob builds and package exports, the example app, threading and error conventions, Jest mocks, Expo config plugins, versioning and publishing.
---

# Library Authoring

## Scaffold

```bash
npx create-react-native-library@latest react-native-device-info-lite
```

The prompts choose:

| Prompt | Options |
|--------|---------|
| Library type | `turbo-module`, `fabric-view`, `nitro-module`, `nitro-view`, `library` (JS only) |
| Native languages | `kotlin-objc`, `kotlin-swift` (Nitro), `cpp`, `js` |
| Example app | `vanilla` (Community CLI), `expo`, `test-app` (Microsoft) |
| Tools | ESLint/Prettier, Lefthook + commitlint, release-it, Jest, Turborepo cache, web support |

Inside an existing app the CLI offers a **local library** (`--local`): it lands in `modules/` and is linked through the app's `package.json`, so it autolinks without publishing.

Generated layout:

```
react-native-device-info-lite/
├── src/
│   ├── NativeDeviceInfoLite.ts   # Codegen spec
│   └── index.tsx                 # public API
├── android/                      # Gradle module, Kotlin sources
├── ios/                          # Objective-C++ sources
├── example/                      # app that consumes the library via the workspace
├── DeviceInfoLite.podspec
└── package.json                  # codegenConfig, exports, bob config
```

## Build and package exports

`react-native-builder-bob` compiles `src/` to `lib/` (ES modules + TypeScript declarations):

```json
{
  "main": "./lib/module/index.js",
  "types": "./lib/typescript/src/index.d.ts",
  "exports": {
    ".": {
      "source": "./src/index.tsx",
      "types": "./lib/typescript/src/index.d.ts",
      "default": "./lib/module/index.js"
    },
    "./package.json": "./package.json"
  },
  "files": ["src", "lib", "android", "ios", "cpp", "*.podspec", "!**/__tests__", "!android/build", "!ios/build"],
  "react-native-builder-bob": {
    "source": "src",
    "output": "lib",
    "targets": [["module", { "esm": true }], ["typescript", { "project": "tsconfig.build.json" }]]
  }
}
```

- Ship `src/` too: Codegen reads the TypeScript spec from `jsSrcsDir` in the consumer's build.
- The source condition (the scaffold picks its name) lets the example app and monorepos consume TypeScript directly.
- Keep `peerDependencies` on `react` and `react-native` (`"*"` or a wide range); never list them as dependencies.

## Develop with the example app

```bash
yarn                      # installs the workspace (library + example)
yarn example start        # Metro for the example
yarn example android      # or: yarn example ios
yarn typecheck && yarn lint && yarn test
```

Edit `src/` and reload. Native changes need a rebuild of the example app; Codegen spec changes need `pod install` (iOS) and a clean Gradle build.

## Native conventions

**Threading**

| Platform | Default | Do |
|----------|---------|-----|
| Android | Module methods run on the JS thread (sync) or the native modules thread | Post UI work with `runOnUiThread` / `UiThreadUtil.runOnUiThread`; run I/O on an executor or coroutine scope you cancel in `invalidate()` |
| iOS | Methods run on the module's method queue | Dispatch UIKit work to `dispatch_get_main_queue()`; heavy work to a background queue |

**Errors**: reject promises with a stable code and message (`promise.reject("E_PERMISSION", "Camera permission denied")` / `reject(@"E_PERMISSION", @"…", nil)`), document the codes, and map them to typed JS errors in `index.tsx`.

**Lifecycle**: release listeners, sensors and timers in `invalidate()` (both platforms) so reloads and multiple React instances don't leak.

**Events**: start native observation when the first JS listener subscribes and stop when the last unsubscribes.

## Testing

- **JS layer**: mock the native module in Jest and test the public API.
  ```ts
  // __mocks__ or jest.setup.ts in consuming apps
  jest.mock('react-native-device-info-lite/src/NativeDeviceInfoLite', () => ({
    __esModule: true,
    default: { getModel: () => 'Pixel 9', getBatteryState: async () => ({ level: 1, charging: false }) },
  }));
  ```
  Ship a ready-made mock (`react-native-device-info-lite/jest`) so consumers don't write their own.
- **Native layer**: JUnit/Robolectric in `android/src/test`, XCTest via the example app's test target.
- **Integration**: run the example app in CI on both platforms (clean builds catch Codegen and linking errors).

## Expo support

Libraries with native code work in Expo dev builds automatically through autolinking. When the library needs native configuration (permissions, `Info.plist` keys, Gradle settings), ship a config plugin:

```js
// app.plugin.js
const { withInfoPlist, withAndroidManifest, AndroidConfig } = require('expo/config-plugins');

module.exports = function withDeviceInfoLite(config, { motionUsage } = {}) {
  config = withInfoPlist(config, (c) => {
    c.modResults.NSMotionUsageDescription ??= motionUsage ?? 'Allow $(PRODUCT_NAME) to access motion data';
    return c;
  });
  return AndroidConfig.Permissions.withPermissions(config, ['android.permission.ACTIVITY_RECOGNITION']);
};
```

Users add it to `app.json` `plugins`. Note in the README that the library doesn't run in Expo Go.

## Versioning and publishing

- Semver: a spec change that removes or renames a method is a **major** version — consumers' native builds break.
- Declare the minimum React Native version you test against; test the latest two RN minors in CI.
- Release with `release-it` (scaffolded) or Changesets; publish with provenance from CI (`npm publish --provenance`).
- Add the library to React Native Directory with its platform and New Architecture support.

## Key Points

- `create-react-native-library` gives you Codegen, podspec, Gradle, example app and builder-bob wired correctly; `--local` for in-app modules.
- Ship `src/` with the spec; `react`/`react-native` as peer dependencies.
- UI on the main thread, I/O off it; clean up in `invalidate()`; stable error codes.
- Mock the native module for Jest, run the example app in CI, ship a config plugin for Expo users.
