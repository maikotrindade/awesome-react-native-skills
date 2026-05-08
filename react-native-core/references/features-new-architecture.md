---
name: react-native-new-architecture
description: React Native New Architecture overview — Fabric renderer, JSI, TurboModules. Enabled by default since RN 0.76. What changed and what it means for your app code.
---

# New Architecture

React Native's New Architecture replaces the old JSON bridge with three components: **JSI**, **Fabric**, and **TurboModules**. It is **enabled by default since RN 0.76** (released late 2024).

---

## What Changed

| Old Architecture | New Architecture |
|-----------------|-----------------|
| Async JSON serialization bridge | JSI — direct C++ bindings, synchronous calls |
| Shadow tree synced from JS thread | Fabric — layout on UI thread, synchronous |
| All native modules loaded at startup | TurboModules — lazy, Codegen-verified |
| React 18 concurrent features blocked | Full concurrent mode support |
| `useNativeDriver: true` opt-in needed | All Reanimated v3 runs on UI thread by default |

---

## JSI (JavaScript Interface)

JSI replaces the JSON bridge with direct C++ function calls between JS and native code. Key implications:

- Native functions can be **called synchronously** from JS
- JS objects are **passed by reference** (no serialization)
- Powers MMKV's synchronous API, Reanimated's worklets, and TurboModules

No changes needed in your JS code — JSI is transparent unless you're writing native modules.

---

## Fabric Renderer

Fabric is the new native rendering system. It unifies the JavaScript shadow tree with the native view tree and runs layout on the UI thread.

**What changes for app code:**

```tsx
// useLayoutEffect fires synchronously (same as React DOM behavior)
useLayoutEffect(() => {
  // measurements are available immediately — no async round-trip
  const node = ref.current;
}, []);

// measure() returns synchronously via Reanimated v3
import { measure, useAnimatedRef } from 'react-native-reanimated';
const aref = useAnimatedRef<Animated.View>();
runOnUI(() => {
  const layout = measure(aref); // synchronous on UI thread
})();
```

React 18 concurrent features now work as expected:

```tsx
import { startTransition, useDeferredValue, Suspense } from 'react';

// Non-urgent state updates — won't block user input
startTransition(() => setSearchQuery(text));

// Defer expensive renders
const deferred = useDeferredValue(query);
```

---

## TurboModules

TurboModules replace bridge-based native modules. They are:
- **Lazy-loaded** — module code doesn't initialize until first call
- **Codegen-verified** — TypeScript spec is used to generate native boilerplate
- **Interoperable** — old modules still work via the interop layer

Most app developers interact with TurboModules only through third-party libraries. You need to know about them if you write a custom native module.

### TurboModule TypeScript Spec

```ts
// NativeMyModule.ts
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  getValue(key: string): string | null;
  setValue(key: string, value: string): void;
}

export default TurboModuleRegistry.getEnforcing<Spec>('MyModule');
```

---

## Verify New Architecture Is Active

```tsx
// In your app (development only)
import { TurboModuleRegistry } from 'react-native';
console.log('New Arch active:', !!TurboModuleRegistry.get('DevSettings'));
```

In build config:

```properties
# android/gradle.properties
newArchEnabled=true
```

```ruby
# ios/Podfile
ENV['RCT_NEW_ARCH_ENABLED'] = '1'
```

---

## Third-Party Library Compatibility

Most popular libraries support the New Architecture. Check [reactnative.directory](https://reactnative.directory) and filter by "New Architecture".

Libraries confirmed compatible:
- React Navigation v6+
- React Native Reanimated v3+
- React Native Gesture Handler v2+
- MMKV
- React Native Firebase v18+
- React Native Screens v3+

**Interop Layer**: Old-style bridge modules still function in RN 0.74+ via the compatibility layer — they don't need immediate migration.

---

## LayoutAnimation on New Architecture (Android)

`LayoutAnimation` is **not compatible** with New Architecture on Android. Use Reanimated v3 `Layout` transitions instead:

```tsx
// Old — breaks on Android with New Architecture
LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut);
setExpanded(true);

// New — works everywhere
import Animated, { Layout } from 'react-native-reanimated';
<Animated.View layout={Layout.springify()}>
```

---

## Concurrent React in Practice

```tsx
import { Suspense, startTransition } from 'react';

function SearchScreen() {
  const [input, setInput] = useState('');
  const [query, setQuery] = useState('');

  const handleChange = (text: string) => {
    setInput(text);           // urgent: input feels instant
    startTransition(() => {
      setQuery(text);         // non-urgent: results can wait
    });
  };

  return (
    <>
      <TextInput value={input} onChangeText={handleChange} />
      <Suspense fallback={<ActivityIndicator />}>
        <SearchResults query={query} />
      </Suspense>
    </>
  );
}
```
