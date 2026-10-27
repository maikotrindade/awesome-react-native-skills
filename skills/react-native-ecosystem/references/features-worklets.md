# Features — React Native Worklets

`react-native-worklets` is the threading layer under Reanimated 4, split into its own package. It lets you run JavaScript functions ("worklets") on the UI thread or on background runtimes, and move data between them.

> Targets `react-native-worklets` 0.13 (React Native 0.86–0.88, New Architecture only). Reanimated 4 lists it as a required peer dependency, so most apps already have it.

## Installation

```bash
npx expo install react-native-worklets   # Expo — Babel plugin is applied by babel-preset-expo
npm install react-native-worklets         # bare RN
```

Bare projects add `'react-native-worklets/plugin'` as the **last** Babel plugin. The plugin finds functions marked with `'worklet'` (and the callbacks of Reanimated hooks, which are workletized automatically) and makes them transferable to other runtimes.

## Runtimes

| Runtime | Where it runs | Typical use |
|---------|---------------|-------------|
| RN runtime | JS thread (React) | Components, state, business logic |
| UI runtime | Main/UI thread | Animations, gesture callbacks, synchronous layout reads |
| Worker runtimes | Background threads you create | CPU-heavy work that must not block JS or UI |

Check where code is running with `getRuntimeKind()`, `isRNRuntime()`, `isUIRuntime()`, or `isWorkerRuntime()`.

## Scheduling API

| Function | Direction | Returns |
|----------|-----------|---------|
| `scheduleOnUI(fn, ...args)` | RN → UI | `void` (fire-and-forget) |
| `runOnUIAsync(fn, ...args)` | RN → UI | `Promise<result>` |
| `runOnUISync(fn, ...args)` | RN → UI | result (blocks the JS thread) |
| `scheduleOnRN(fn, ...args)` | UI/worker → RN | `void` |
| `scheduleOnRuntime(rt, fn, ...args)` | any → worker | `void` |
| `runOnRuntimeAsync(rt, fn, ...args)` | any → worker | `Promise<result>` |
| `runOnRuntimeSync(rt, fn, ...args)` | any → worker | result (blocks caller) |

`runOnUI(fn)(...)` and `runOnJS(fn)(...)` are the deprecated curried forms. The new functions take arguments directly.

```ts
import { scheduleOnUI, scheduleOnRN, runOnUIAsync } from 'react-native-worklets';
import { measure } from 'react-native-reanimated';

// fire-and-forget on the UI thread
scheduleOnUI(() => {
  'worklet';
  sv.value = withSpring(1);
});

// read UI-thread layout from JS without blocking
const layout = await runOnUIAsync(() => {
  'worklet';
  return measure(aref);
});

// call back into React from a gesture or animation callback
scheduleOnRN(setSelectedId, id);
```

Functions passed to `scheduleOnRN` do **not** need a `'worklet'` directive. They are plain JS functions that run on the RN runtime.

## Background Work on a Worker Runtime

```ts
import { createWorkletRuntime, runOnRuntimeAsync } from 'react-native-worklets';

const parserRuntime = createWorkletRuntime({ name: 'parser' });

export async function parseLargePayload(raw: string) {
  return runOnRuntimeAsync(parserRuntime, (input: string) => {
    'worklet';
    return JSON.parse(input).items.map((i: { id: string }) => i.id);
  }, raw);
}
```

- Create runtimes once (module scope), not per call. Each runtime has its own JS heap.
- Worker runtimes include an event loop (`setTimeout`, `setInterval`, `requestAnimationFrame`) by default. Turn it off with `enableEventLoop: false` if you don't need it.
- Use `initializer` in the config to set up globals once when the runtime starts.
- Worker runtimes have no React, no DOM, and no native modules unless those modules expose JSI bindings.

## Moving Data Between Runtimes

Values captured by a worklet are **copied** into the target runtime when it is scheduled. That works for primitives, plain objects, arrays, `ArrayBuffer`s, and other worklets, but:

- Class instances lose their prototype. Use `registerCustomSerializable` to teach worklets how to copy your own types.
- Large captured objects are copied on every scheduling call. Pass IDs or slices instead.
- Mutating a copied object on one runtime does not affect the other.

For state that must be **shared** rather than copied, use a Synchronizable:

```ts
import { createSynchronizable } from 'react-native-worklets';

const progress = createSynchronizable(0);

// worker runtime
scheduleOnRuntime(rt, () => {
  'worklet';
  for (let i = 1; i <= 100; i++) {
    doChunk(i);
    progress.setBlocking(i);
  }
});

// RN runtime (e.g., polled by a timer)
const current = progress.getDirty();   // fast read; may be slightly stale
const exact = progress.getBlocking();  // waits for the lock
```

| Method | Behavior |
|--------|----------|
| `getDirty()` / `setDirty(v)` | No locking: fastest, last write wins |
| `getBlocking()` / `setBlocking(v)` | Takes the lock; `setBlocking` accepts an updater `(prev) => next` |
| `lock()` / `unlock()` | Hold the lock across several operations |

Reanimated's `useSharedValue` is the right tool for values that drive animations. Use Synchronizables only for cross-thread coordination outside animations.

## Third-Party Code Inside Worklets (Bundle Mode)

By default a worklet can only call other worklets and the globals available on its runtime. To `import` npm libraries inside worklets on worker runtimes, enable **Bundle Mode**. It needs a Babel plugin option and a Metro config change, so follow the worklets documentation for your package manager. Check `isBundleModeEnabled()` at runtime to confirm it is active.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Calling a React setter directly inside a UI worklet | Wrap it: `scheduleOnRN(setState, value)` |
| Forgetting `'worklet'` in a function passed to `scheduleOnUI` or a worker runtime | Add the directive as the function's first statement |
| Creating a runtime inside a component body | Create it once at module scope |
| Blocking JS with `runOnUISync` / `runOnRuntimeSync` in render | Use the async variants or effects |
| Capturing a huge object in a frequently scheduled worklet | Capture an ID and look it up on the target runtime |
| Using Reanimated 3's `runOnJS` import path | Import scheduling helpers from `react-native-worklets` |
