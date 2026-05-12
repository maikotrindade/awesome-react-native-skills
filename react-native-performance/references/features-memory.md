---
name: react-native-performance-memory
description: Detecting and fixing memory leaks in React Native — JS useEffect cleanup, native reference cycles, JNI refs, and smart-pointer patterns in C++/Swift/Kotlin.
---

# Memory & Leak Hunting

Memory leaks come from three places: JS (uncleaned subscriptions/timers/closures), native (reference cycles, missing `delete`, unremoved listeners), and the JS↔native boundary (Turbo Modules not cancelling work on invalidate). Detect with profilers, fix with discipline at cleanup hooks.

## Detection — JS Side

React Native DevTools → **Memory** tab → "Allocation instrumentation on timeline" → Start → exercise the suspect flow → Stop.

**Reading the timeline:**

- Blue bars: allocated
- Gray bars: collected
- **Blue bars that stay blue after GC = leak**

| Column | Meaning |
|--------|---------|
| Constructor | Object type |
| Count | Live instances |
| Shallow size | Memory of the object itself |
| Retained size | Memory freed if object is deleted |

**Red flag:** large retained size with small shallow size = closures holding large objects.

Force a GC (allocate something disposable) before concluding there's a leak.

## Detection — Native Side

**iOS — Xcode Instruments → Leaks** (`Cmd+I`, pick the Leaks template). Red markers = leak. The "Responsible Frame" column points at the function.

For a quick check, use the **Debug Navigator → Memory** graph during normal use.

**Android — Android Studio Profiler → Track Memory Consumption.** If `allocations >> deallocations`, you have a leak. Rotate the device multiple times (config change) — old activities should be freed.

> RN apps usually set `android:configChanges` to opt out of activity recreation, but native code attached to activities might still leak.

## Common JS Leak Patterns

**1. useEffect missing cleanup** (most common):

```jsx
useEffect(() => {
  const sub = EventEmitter.addListener('event', handler);
  return () => sub.remove();    // REQUIRED
}, []);

useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id);
}, []);
```

**2. Closures capturing large objects:**

```jsx
// BAD — captures entire `this.largeData`
createLeakyFn() { return () => this.largeData.length; }

// GOOD — extract the primitive
createFn() {
  const length = this.largeData.length;
  return () => length;
}
```

**3. Growing global arrays / module-level caches:**

```jsx
let cache = [];
const add = () => { cache.push(generateLargeData()); };  // never shrinks
```

Use `WeakRef`/`WeakMap` for caches, or clear explicitly.

## Common Native Leak Patterns

**Reference cycle (Swift):**

```swift
class Parent { var child: Child? }
class Child  { weak var parent: Parent? }   // weak breaks the cycle
```

**Reference cycle (C++):** mirror with `std::weak_ptr` on the back-pointer.

```cpp
class A { std::shared_ptr<B> b; };
class B { std::weak_ptr<A>   a; };
```

**Missing `delete` (C++)** — fix with RAII:

```cpp
auto data = std::make_unique<LargeData>();   // auto-freed on scope exit
```

**Unremoved listener (Android):**

```kotlin
override fun onCreate(...) { EventManager.addListener(this) }
override fun onDestroy() { EventManager.removeListener(this); super.onDestroy() }
```

**Singleton holding strong refs** — use `WeakReference`:

```kotlin
object Cache { private val items = mutableMapOf<String, WeakReference<Callback>>() }
```

## Turbo Module Cleanup

Async work started in a module *must* be cancelled in `invalidate()` / `RCTInvalidating`, otherwise it survives Metro reloads and leaks JS context.

```kotlin
class MyModule : NativeMyModuleSpec(reactContext) {
  private val scope = CoroutineScope(Dispatchers.Default + SupervisorJob())
  override fun invalidate() { super.invalidate(); scope.cancel() }
}
```

iOS: implement `RCTInvalidating` and tear down observers, queues, timers in `invalidate()`.

## Memory Management Models

| Pattern | Languages | Mechanism |
|---------|-----------|-----------|
| Reference counting | Swift, Obj-C, C++ smart pointers | Free at refcount = 0 |
| Garbage collection | Kotlin/Java, JavaScript | GC scans unreachable |
| Manual | C, raw C++ pointers | `new` / `delete` |

### C++ Smart Pointers — Pick the Right One

```cpp
auto p = std::make_unique<T>(args);   // single owner; move-only
auto s = std::make_shared<T>(args);   // shared owner; refcount
std::weak_ptr<T> w = s;               // observer; doesn't extend lifetime
if (auto live = w.lock()) { /* still alive */ }
```

Pass `shared_ptr` by `const&` when you don't need to extend lifetime — avoids the atomic refcount bump.

### Swift ARC

- `strong` (default): keeps object alive
- `weak`: nilled when target deallocated; use for delegates, parents, observer back-refs
- `unowned`: like `weak` but non-optional; crashes if target gone — only when lifetime is guaranteed

### Kotlin — `WeakReference` and `AutoCloseable`

```kotlin
class MyResource : AutoCloseable {
  private val listener = object : Callback { override fun onEvent() {} }
  init { EventManager.addListener(listener) }
  override fun close() { EventManager.removeListener(listener) }
}
```

For caches of GC-able keys: `WeakHashMap`.

### Swift `Unmanaged` (C interop)

Match `passRetained` ↔ `takeRetainedValue`, `passUnretained` ↔ `takeUnretainedValue`. Mismatches leak or over-release.

## Best Practices Summary

| Language | Rule |
|----------|------|
| C++ | `unique_ptr` / `shared_ptr` / `weak_ptr` — avoid raw `new` |
| Swift | `weak` on delegate and parent back-refs |
| Kotlin | `AutoCloseable` + `WeakReference` for listener caches |
| JS / RN | Always return a cleanup from `useEffect`; cancel timers and subscriptions |

## Verification

After fixing, re-profile the same flow:

- iOS Leaks: no red markers
- Android: allocations ≈ deallocations after GC
- DevTools timeline: blue bars eventually turn gray

## Key Points

- Most RN leaks are missing `useEffect` cleanup — subscriptions, timers, intervals. Always return a teardown function.
- "Retained size large, shallow size small" in the JS heap profiler = closure holding a big object.
- Native reference cycles need `weak` (Swift) or `weak_ptr` (C++) on the back-pointer.
- Turbo Modules must cancel coroutines / invalidate observers in `invalidate()` — otherwise they survive Metro reloads.
- Test Android leaks via repeated config changes (rotation); test iOS leaks in release builds — some only appear there.
- Prefer RAII (C++), ARC + `weak` (Swift), `AutoCloseable` + `WeakReference` (Kotlin) over manual lifecycle tracking.

