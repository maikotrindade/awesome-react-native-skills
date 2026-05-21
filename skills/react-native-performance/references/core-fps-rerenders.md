---
name: react-native-performance-fps-rerenders
description: Measure FPS, profile React re-renders, and apply Reanimated worklets, atomic state, concurrent React, and the React Compiler to hit 60 FPS.
---

# FPS, Re-renders & Runtime Performance

Deep performance work for the JS thread: how to measure, profile, and apply the right tool (worklets, atomic state, deferred values, compiler) to keep frames smooth.

## Measure FPS

| Tool | Platform | What it gives you |
|------|----------|-------------------|
| Built-in Perf Monitor | iOS + Android | UI thread FPS, JS thread FPS, RAM (overlay) |
| Flashlight | Android only | Score (0-100), avg FPS, FPS graph, CPU/RAM, JSON export |

Open Perf Monitor: Dev Menu (Cmd+Ctrl+Z iOS / Cmd+M Android) -> "Perf Monitor".

```bash
# Flashlight (install from official channel only — never pipe curl|sh)
flashlight measure
flashlight measure --output results.json
flashlight compare baseline.json current.json
```

**Always disable dev mode** before measuring — JS Dev Mode skews everything:

```bash
# Expo
npx expo start --no-dev --minify

# React Native CLI: build release variant
```

### Reading the two threads

| Symptom | Likely cause |
|---------|--------------|
| UI FPS drops, JS FPS fine | Native rendering (too many views, heavy native animation) |
| JS FPS drops, UI FPS fine | JS computation, expensive re-renders, blocking work |
| Both drop | Mixed — start with JS profiling |

| FPS | Perception | Action |
|-----|-----------|--------|
| 55-60 | Smooth | Ship |
| 45-55 | Slight stutter | Investigate |
| 30-45 | Noticeable jank | Optimize |
| <30 | Choppy | Critical fix |

Frame budgets: 60 FPS = 16.6ms, 120 FPS = 8.3ms per frame.

## Profile React Re-renders

Open React Native DevTools — press `j` in the Metro terminal, or shake -> "Open DevTools".

**Profiler workflow:**

1. Profiler tab -> gear icon -> enable "Record why each component rendered while profiling"
2. "Start profiling" (or "Reload and start profiling" for startup)
3. Drive the actual flow under review (not just idle)
4. Stop -> inspect commit timeline, re-render counts, heaviest commit

**Flame graph colors:** yellow = slowest, green = fast/memoized, gray = did not render. Right panel shows "Why did this render?" (which prop changed, or parent re-rendered).

Use the **Ranked** tab for bottom-up sort by render time. For non-React perf, use the **JavaScript Profiler** tab and its "Heavy (Bottom Up)" view.

| Symptom | Cause | Fix |
|---------|-------|-----|
| Many yellow components | Cascading re-renders | Memoize or enable React Compiler |
| "Props changed" on a callback | Inline function recreated | `useCallback` (or compiler) |
| "Parent component rendered" | State too high in tree | Move state down, or atomic state |
| Long JS thread block | Heavy computation | Defer with `useDeferredValue`, move off main path |

Only add memoization where the profiler proves it matters — don't infer stale closures from a snippet.

## React Compiler (auto-memoization)

Opt-in via **Expo SDK 54+** — sets `experiments.reactCompiler: true` in `app.json`.

```bash
npx react-compiler-healthcheck@latest     # check Rules-of-React compliance first
```

**Expo SDK 54+:**

```bash
npx expo install babel-plugin-react-compiler
```

```json
// app.json
{ "expo": { "experiments": { "reactCompiler": true } } }
```

**Expo SDK 52-53:**

```bash
npx expo install babel-plugin-react-compiler@beta react-compiler-runtime@beta
```

**React Native (no Expo):**

```javascript
// babel.config.js — compiler must run FIRST
const ReactCompilerConfig = { target: '19' }; // '18' for RN < 0.78

module.exports = (api) => {
  api.cache(true);
  return {
    presets: ['module:@react-native/babel-preset'],
    plugins: [['babel-plugin-react-compiler', ReactCompilerConfig]],
  };
};
```

ESLint plugin: `eslint-plugin-react-compiler` (Expo) or `eslint-plugin-react-hooks@latest` `recommended-latest` preset (bare). Optimized components show a `Memo ✨` badge in React DevTools.

### Incremental adoption

```javascript
// Limit to a directory
const ReactCompilerConfig = {
  target: '19',
  sources: (filename) => filename.includes('src/feature/'),
};
```

```jsx
// Opt out one component
function Problematic() {
  'use no memo';
  return <Text>...</Text>;
}
```

After config changes, restart Metro with `--clear` / `--reset-cache`.

### What breaks compilation

```jsx
// Mutating props
items.push('new');

// Mutating state during render
const [items, setItems] = useState([]);
items.push('new');

// Non-idempotent render (module-scope mutation)
let counter = 0;
const Bad = () => { counter++; return <Text>{counter}</Text>; };
```

Class components are not optimized. Compiler skips files with ESLint errors — fix those first to actually get optimization.

## Atomic State (Jotai / Zustand)

Context re-renders every consumer on any state change. Atomic libraries subscribe per-atom/selector — only changed pieces re-render. Use when React Compiler isn't available, or for global state.

**Jotai:**

```jsx
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

const filterAtom = atom('all');
const todosAtom = atom([]);

// Derived
const filteredAtom = atom((get) => {
  const f = get(filterAtom);
  const t = get(todosAtom);
  return f === 'all' ? t : t.filter((x) => x.completed === (f === 'completed'));
});

const FilterMenu = () => {
  const [filter, setFilter] = useAtom(filterAtom); // only re-renders on filter
};

const TodoItem = ({ id }) => {
  const setTodos = useSetAtom(todosAtom); // setter only — no read subscription
};
```

**Zustand:** *always* use selectors, otherwise the whole store subscribes.

```jsx
import { create } from 'zustand';

const useTodoStore = create((set, get) => ({
  filter: 'all',
  todos: [],
  setFilter: (filter) => set({ filter }),
  toggleTodo: (id) => set((s) => ({
    todos: s.todos.map((t) => t.id === id ? { ...t, completed: !t.completed } : t),
  })),
}));

// Selector — re-renders only when filter changes
const filter = useTodoStore((s) => s.filter);
```

| | Context | Jotai | Zustand |
|---|---------|-------|---------|
| Re-render scope | All consumers | Atom subscribers | Selector subscribers |
| Derived state | Manual | Built-in atoms | Selectors |
| Bundle | 0 KB | ~3 KB | ~2 KB |

Don't over-atomize — group related state.

## Concurrent React

Requires New Architecture (default in RN 0.76+). Use `useDeferredValue` for a single expensive consumer; `useTransition` when you have multiple state updates and want loading state.

```jsx
// useDeferredValue — input stays responsive
const [query, setQuery] = useState('');
const deferredQuery = useDeferredValue(query);

<TextInput value={query} onChangeText={setQuery} />
<ExpensiveList query={deferredQuery} />

// Stale indicator
const isStale = query !== deferredQuery;
```

```jsx
// useTransition — multiple updates, low-priority work has isPending
const [count, setCount] = useState(0);
const [isPending, startTransition] = useTransition();

const onPress = () => {
  setCount(c => c + 1);                       // high priority
  startTransition(() => setHeavy(compute())); // low priority, interruptible
};
```

**Critical:** `useDeferredValue` is useless if the child re-renders from the parent anyway — wrap expensive children in `memo()` (or rely on React Compiler).

```jsx
const SlowComponent = memo(({ query }) => { /* expensive */ });
```

| Scenario | Hook |
|----------|------|
| Single value drives expensive render | `useDeferredValue` |
| Multiple updates, some non-critical | `useTransition` |
| Need loading state for transition | `useTransition` (`isPending`) |
| Suspense + search | `Suspense` + `useDeferredValue` |

These hooks don't make code faster — they reprioritize when work runs.

## Reanimated (UI-thread animations)

**Reanimated v3 is stable. v4 is opt-in/beta and requires New Architecture + `react-native-worklets`.**

The JS-thread `Animated` API drops frames whenever JS is busy. Reanimated runs animations on the UI thread via worklets — smooth even during heavy JS work.

```jsx
import Animated, { useSharedValue, useAnimatedStyle, withTiming } from 'react-native-reanimated';

const opacity = useSharedValue(0);
const animatedStyle = useAnimatedStyle(() => ({ opacity: opacity.value })); // UI thread

useEffect(() => { opacity.value = withTiming(1, { duration: 500 }); }, []);

return <Animated.View style={[styles.box, animatedStyle]} />;
```

### Crossing threads (v4 / worklets package)

```jsx
import { scheduleOnUI, scheduleOnRN } from 'react-native-worklets';

scheduleOnUI(() => { 'worklet'; /* UI-thread work */ });

const animatedStyle = useAnimatedStyle(() => {
  if (progress.value === 1) {
    scheduleOnRN(trackAnalytics, progress.value); // call JS from worklet
  }
  return { opacity: progress.value };
});
```

### v3 -> v4 migration

```bash
npm install react-native-reanimated react-native-worklets
```

```javascript
// babel.config.js — must be LAST
module.exports = { plugins: ['react-native-worklets/plugin'] };
```

| v3 | v4 | Package |
|----|----|---------|
| `runOnUI(fn)()` | `scheduleOnUI(fn)` | `react-native-worklets` |
| `runOnJS(fn)(args)` | `scheduleOnRN(fn, args)` | `react-native-worklets` |
| `executeOnUIRuntimeSync` | `runOnUISync` | `react-native-worklets` |
| `useScrollViewOffset` | `useScrollOffset` | `react-native-reanimated` |
| `useWorkletCallback` | `useCallback` + `'worklet';` | React |

Removed: `useAnimatedGestureHandler` (use Gesture API), `addWhitelisted*Props`, `combineTransition`.

`withSpring` v4: `energyThreshold` replaces both `restDisplacementThreshold` and `restSpeedThreshold`; `duration` is now perceptual (~1.5x actual).

**Worklet pitfalls:** keep worklets fast (no heavy computation inside `useAnimatedStyle`); use `useSharedValue`, not `useState`; remember the `'worklet'` directive on inline worklet functions; animate `Animated.View` etc., not plain `View`.

## Key Points

- Measure first: Perf Monitor for a quick read, Flashlight for Android benchmarks with CI-friendly JSON output. Always disable dev mode.
- Profile re-renders with the "Why did this render?" panel — don't add `useCallback`/`memo` speculatively.
- React Compiler (Expo SDK 54+, `experiments.reactCompiler: true`) replaces most manual memoization. Run the healthcheck first and fix ESLint errors — the compiler skips files that fail.
- Use atomic state (Jotai/Zustand selectors) to fix Context-wide re-renders without manual memoization. Always use selectors with Zustand.
- `useDeferredValue` / `useTransition` require New Arch + memoized children to actually help. They reprioritize, not accelerate.
- Reanimated v3 is stable; v4 needs New Arch and `react-native-worklets`. Migrate JS-thread `Animated` code to UI-thread shared values for animations that stay smooth under JS load.
