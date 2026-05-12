---
name: react-native-performance-native
description: Native-side performance — Turbo Module threading, sync vs async, view flattening, native SDKs over polyfills, 16 KB alignment, platform tooling.
---

# Native Performance

Optimizations that live on the native side: threading discipline, view-hierarchy flattening, replacing JS polyfills with native SDKs, and Android 16 KB page-size alignment. For the broader New Architecture overview, see `react-native-core/features-new-architecture.md`.

## Threading Model

| Action | iOS Thread | Android Thread |
|--------|------------|----------------|
| Module init | Main | JS (lazy) / Native (eager) |
| Sync method | JS | JS |
| Async method | Native modules | Native modules |
| View init / prop update | Main | Main |
| Yoga layout | JS | JS |
| Invalidate | Native modules | ReactHost pool |

**Rule:** sync methods block the JS thread. Keep them under 16 ms or make them async.

Thread names in debugger: `mqt_v_js` (JS), `mqt_v_native` (native modules), Main thread (UI).

## Turbo Modules — Sync vs Async

Bad — blocks JS for 2s:

```swift
@objc func heavyWork() -> NSNumber {
  Thread.sleep(forTimeInterval: 2)
  return 42
}
```

Good — async on background queue:

```swift
@objc func heavyWork(
  _ input: Double,
  resolve: @escaping RCTPromiseResolveBlock,
  reject: RCTPromiseRejectBlock
) {
  DispatchQueue.global(qos: .userInitiated).async {
    let result = self.compute(input)
    resolve(result)
  }
}
```

Android — coroutines, cancel scope on invalidate:

```kotlin
class MyModule(reactContext: ReactApplicationContext) : NativeMyModuleSpec(reactContext) {
  private val scope = CoroutineScope(Dispatchers.Default + SupervisorJob())

  override fun heavyOperation(input: Double, promise: Promise?) {
    scope.launch {
      try {
        val result = withContext(Dispatchers.Default) { compute(input) }
        promise?.resolve(result)
      } catch (e: Exception) { promise?.reject("ERROR", e.message) }
    }
  }

  override fun invalidate() { super.invalidate(); scope.cancel() }
}
```

### C++ Turbo Modules (skip JNI)

C++ Turbo Modules give JS direct C++ function references via JSI — no JNI lookup per call. Use for cross-platform hot paths.

```cpp
namespace facebook::react {
class MyCppModule : public TurboModule {
public:
  MyCppModule(std::shared_ptr<CallInvoker> jsInvoker);
  double multiply(double a, double b);
};
}
```

Register for iOS auto-linking with `registerCxxModuleToGlobalModuleMap` in a `+load` method.

### Language Interop Costs

| Interface | Overhead | Notes |
|-----------|----------|-------|
| Obj-C ↔ C++ | ~0 | Compile-time |
| Swift ↔ C++ | ~0 | Swift 5.9+ interop |
| Kotlin ↔ C++ (JNI) | Medium | Per-call lookup |
| C++ Turbo Module | Low | JSI direct |

### Android Eager Loading

Default Android Turbo Modules init lazily on the JS thread. For modules needed during startup, set `needsEagerInit = true`:

```kotlin
ReactModuleInfo(NAME, NAME, false, /* needsEagerInit */ true, false, true)
```

## View Flattening

The renderer removes "layout-only" views (no `backgroundColor`, `borderWidth`, `shadow`, events, `opacity < 1`, `overflow: hidden`). Good for memory, but it breaks native components that expect a specific child count.

```tsx
<NativeTabBar>
  <Tab1 collapsable={false} />
  <Tab2 collapsable={false} />
</NativeTabBar>
```

`collapsable={false}` is the cleanest way to opt out. Mapping to native:

- iOS: `<View />` → `RCTViewComponentView`, `<Text />` → `RCTTextView`
- Android: `<View />` → `ReactViewGroup`, `<Text />` → `ReactTextView`

Inspect with Xcode "Debug View Hierarchy" or Android Studio Layout Inspector.

## Native SDKs Over JS Polyfills

JS polyfills bloat the bundle and slow runtime. Audit Hermes support first, then strip what's covered natively.

### Hermes Intl Coverage

| API | Hermes | Keep polyfill? |
|-----|--------|----------------|
| `Intl.Collator`, `Intl.DateTimeFormat`, `Intl.getCanonicalLocales`, `Intl.supportedValuesOf` | Yes | No |
| `Intl.NumberFormat` | Partial | Keep if you use `formatToParts()` on iOS |
| `Intl.Locale`, `Intl.PluralRules`, `Intl.RelativeTimeFormat`, `Intl.DisplayNames`, `Intl.ListFormat`, `Intl.Segmenter` | No | Yes |

Removing unneeded `@formatjs/*` polyfills can cut 430+ KB.

### Native Replacements

```bash
npm install react-native-quick-crypto                      # 58x faster than crypto-js
npm install @react-navigation/native-stack react-native-screens
npm install @bottom-tabs/react-navigation react-native-bottom-tabs
```

```tsx
import { createHash } from 'react-native-quick-crypto';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createNativeBottomTabNavigator } from '@bottom-tabs/react-navigation';
```

| Category | Library |
|----------|---------|
| Navigation containers | `react-native-screens` |
| Menus | `zeego` |
| Slider | `@react-native-community/slider` |
| Date picker | `react-native-date-picker` |
| Image caching | `react-native-fast-image` |

## Android 16 KB Page Alignment

Google Play deadline: **Nov 1, 2025** for apps targeting Android 15+. React Native 0.79+ ships aligned, but **third-party `.so` libraries may not be**.

```bash
zipalign -c -P 16 -v 4 app-release.apk
```

Only `arm64-v8a` and `x86_64` are affected. Add to CI:

```bash
zipalign -c -P 16 -v 4 app-release.apk 2>&1 | tee alignment.log
if grep -q "Verification FAILED" alignment.log; then exit 1; fi
```

Trace misaligned libs:

```bash
find node_modules -name "libfoo.so"
```

Alignment cannot be fixed by repackaging — the vendor must rebuild with a 16 KB-compatible toolchain. Test on Pixel 8/8a/9 with "Boot with 16 KB page size" or the 16 KB emulator image.

## Platform Tooling Reference

| Platform | IDE | Deps | Build |
|----------|-----|------|-------|
| iOS | Xcode | CocoaPods | xcodebuild |
| Android | Android Studio | Gradle | Gradle |

```bash
bundle install && cd ios && bundle exec pod install
cd android && ./gradlew clean
xed ios/                                  # open Xcode workspace
npx react-native start --reset-cache      # nuke Metro cache
npx react-native clean                    # full RN clean
```

| Issue | Fix |
|-------|-----|
| Pod install fails | `bundle exec pod install --repo-update` |
| Gradle sync fails | `./gradlew clean` then resync |
| Stale Metro | `--reset-cache` |

## Key Points

- Async Turbo Module methods run on the native modules thread; sync methods block JS — cap them at 16 ms.
- Always cancel coroutine scopes in `invalidate()` to prevent leaks across Metro reloads.
- C++ Turbo Modules avoid per-call JNI overhead via JSI direct refs — best for hot cross-platform paths.
- View flattening removes "layout-only" views; use `collapsable={false}` when native components require an exact child count.
- Audit Hermes Intl support before shipping `@formatjs/*` polyfills; native nav/crypto replacements cut hundreds of KB and run faster.
- Verify 16 KB alignment in CI with `zipalign -P 16`; only 64-bit ABIs are affected.

<!-- Source references: https://github.com/callstackincubator/agent-skills/blob/main/skills/react-native-best-practices/references/native-threading-model.md https://github.com/callstackincubator/agent-skills/blob/main/skills/react-native-best-practices/references/native-turbo-modules.md https://github.com/callstackincubator/agent-skills/blob/main/skills/react-native-best-practices/references/native-view-flattening.md https://github.com/callstackincubator/agent-skills/blob/main/skills/react-native-best-practices/references/native-sdks-over-polyfills.md https://github.com/callstackincubator/agent-skills/blob/main/skills/react-native-best-practices/references/native-android-16kb-alignment.md https://github.com/callstackincubator/agent-skills/blob/main/skills/react-native-best-practices/references/native-platform-setup.md -->
