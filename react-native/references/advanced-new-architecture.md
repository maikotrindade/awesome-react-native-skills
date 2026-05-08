# Advanced — New Architecture

React Native's New Architecture replaces the old bridge with three foundational changes: **JSI**, **Fabric**, and **TurboModules**. Enabled by default since RN 0.76.

---

## What Changed

| Old Architecture | New Architecture |
|-----------------|-----------------|
| Async JSON bridge | JSI — direct C++ bindings, synchronous |
| Shadow tree on JS thread | Fabric — renders natively on UI thread |
| Lazy-loaded bridge modules | TurboModules — lazy, typed, codegen-verified |
| Synchronous `Animated` not possible without `useNativeDriver` | All UI updates synchronous by default |
| React 18 concurrent features blocked | Full concurrent mode support |

---

## Verify New Architecture Is Active

```ts
// Should log "true" in RN 0.76+
import { TurboModuleRegistry } from 'react-native';
console.log('New Arch:', !!TurboModuleRegistry);

// Or check in your app's console
// Bridgeless mode indicator in Metro logs: "Running with bridgeless=true"
```

In `android/gradle.properties`:
```
newArchEnabled=true
```

In `ios/Podfile`:
```ruby
ENV['RCT_NEW_ARCH_ENABLED'] = '1'
```

---

## JSI (JavaScript Interface)

JSI replaces the JSON-over-bridge with direct C++ bindings. Native functions can be called synchronously from JS, and JS objects can be passed as references (not serialized).

**What this means for app developers:**
- Reanimated v3 worklets run directly on the UI thread (already using JSI)
- MMKV synchronous reads are safe
- Custom TurboModules you write can expose synchronous APIs

---

## Fabric (New Renderer)

Fabric is the new native rendering system. It unifies the shadow tree with the native view tree, runs layout calculations on the UI thread, and supports React 18 concurrent features.

**Key changes for app developers:**
- `useLayoutEffect` now fires synchronously (same as web behavior)
- `measure` / `measureInWindow` return values synchronously
- Concurrent features (Suspense, `startTransition`, `useDeferredValue`) work as expected

---

## TurboModules

TurboModules replace old bridge-based native modules. They are:
- **Lazy-loaded** — module code doesn't run until first call
- **Codegen-verified** — TypeScript spec generates native bindings
- **Interoperable** — old modules still work via the interop layer

### Writing a TurboModule

**Step 1 — TypeScript spec**
```ts
// specs/NativeMyModule.ts
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  getValue(key: string): string | null;
  setValue(key: string, value: string): void;
  getConstants(): { defaultTimeout: number };
}

export default TurboModuleRegistry.getEnforcing<Spec>('MyModule');
```

**Step 2 — package.json codegenConfig**
```json
{
  "name": "my-module",
  "codegenConfig": {
    "name": "MyModuleSpec",
    "type": "modules",
    "jsSrcsDir": "specs",
    "android": { "javaPackageName": "com.mymodule" }
  }
}
```

**Step 3 — Run Codegen**
```bash
# Android — runs automatically during gradle build
./gradlew generateCodegenArtifactsFromSchema

# iOS
cd ios && RCT_NEW_ARCH_ENABLED=1 bundle exec pod install
```

**Step 4 — Native implementation** (follows the generated protocol/interface)

---

## Fabric Components (Custom Native Views)

```ts
// specs/MyViewNativeComponent.ts
import type { ViewProps } from 'react-native';
import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';

interface NativeProps extends ViewProps {
  color: string;
  onColorChange: DirectEventHandler<{ color: string }>;
}

export default codegenNativeComponent<NativeProps>('MyView');
```

---

## Third-Party Library Compatibility

Check compatibility before upgrading:

```bash
# reactnative.directory — filter by "New Architecture" support
# Also check each library's GitHub for "fabric" or "turbomodule" tags
```

Libraries confirmed compatible with New Architecture as of 2026:
- React Navigation v6+ / v7
- React Native Reanimated v3+
- React Native Gesture Handler v2+
- MMKV
- React Native Firebase v18+
- React Native Screens v3+

If a library uses the old bridge API, the **Interop Layer** in RN 0.74+ enables it to work alongside New Architecture without code changes (temporary compatibility).

---

## Interop Layer

Old-style modules and components still work in New Architecture via the interop layer:
```ruby
# ios/Podfile — enable interop for old modules
use_react_native!(:hermes_enabled => true, :fabric_enabled => true)
```

Remove the interop layer as libraries migrate:
```gradle
// android/gradle.properties
# Keep until all dependencies are fully migrated
reactNativeArchitectures=arm64-v8a,x86_64
```

---

## Concurrent React Features

With New Architecture + React 18:

```tsx
import { startTransition, useDeferredValue, Suspense } from 'react';

// Non-urgent updates won't block user input
function SearchResults({ query }: { query: string }) {
  const deferredQuery = useDeferredValue(query);

  return (
    <Suspense fallback={<ActivityIndicator />}>
      <ResultsList query={deferredQuery} />
    </Suspense>
  );
}

// Mark state update as non-urgent
function handleInput(text: string) {
  setInputValue(text);               // urgent — update input immediately
  startTransition(() => {
    setSearchQuery(text);            // non-urgent — can be deferred
  });
}
```

---

## Migration Checklist

- [ ] RN 0.76+ (New Architecture on by default)
- [ ] `newArchEnabled=true` in `android/gradle.properties`
- [ ] `RCT_NEW_ARCH_ENABLED=1` in iOS Podfile
- [ ] Verify all native dependencies support New Architecture (reactnative.directory)
- [ ] Replace `Animated` API with Reanimated v3 where possible
- [ ] Test on both platforms — Fabric rendering can surface layout bugs masked by the old renderer
- [ ] Remove `useNativeDriver: true` from Reanimated calls (it's the default now)
