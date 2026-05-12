---
name: react-native-performance-profiling
description: Profiling tools for React Native — DevTools flamegraph, Flashlight FPS, Xcode Instruments, Android Studio Profiler, Perfetto, bundle analyzers.
---

# Profiling Tools & Methodology

Profile the *exact* flow under review on a **release build** running on a **real low-end device** — debug builds, simulators, and dev-mode JS all hide the real cost.

## Tool Map

| Tool | Layer | Best For |
|------|-------|----------|
| RN DevTools Profiler | React | Unnecessary re-renders, "why did this render?" |
| RN DevTools JS Profiler | JS | CPU hotspots in JS code |
| RN DevTools Memory | JS heap | Leaks, retained closures |
| Perf Monitor | UI/JS thread | Quick FPS check |
| Flashlight | Whole-app (Android) | FPS scoring, CI benchmarks |
| Xcode Instruments → Time Profiler | Native iOS | Native CPU hotspots, hangs |
| Xcode Instruments → Leaks | Native iOS | iOS memory leaks |
| Android Studio Profiler | Native Android | CPU, memory, allocations |
| Perfetto | Native Android | Cross-process trace analysis |
| `source-map-explorer` / Emerge xray | Bundle | What's in the JS bundle / app size |

## React DevTools Profiler

Open: press `j` in Metro, or shake → "Open DevTools" → **Profiler** tab.

**Settings (gear icon):** enable "Highlight updates when components render" and "Record why each component rendered while profiling".

Workflow:

1. Start profiling (or "Reload and start profiling" for startup).
2. Drive the audited flow — *not* idle.
3. Stop.
4. Inspect commit timeline, re-render counts, heaviest commit.

**Flame graph colors:**

- Yellow: most render time — focus here
- Green: fast / memoized
- Gray: did not render this commit

The right panel's **"Why did this render?"** is the actionable signal — it names the prop or hook that changed.

| Symptom | Cause | Fix |
|---------|-------|-----|
| Many yellow components | Cascading re-renders | `memo` / React Compiler |
| "Props changed: onPress" | Inline function | `useCallback` |
| "Parent component rendered" | State too high | Move state down / atomic state |
| Long JS block | Heavy computation | Background thread / `useDeferredValue` |

**Ranked view** sorts components by render time (bottom-up).

Only propose memoization/dependency changes when the profiler shows they matter — do not infer stale closures from a snippet alone.

## Measuring FPS

Two FPS streams matter: **UI (main) thread** and **JS thread**. Both must hit the target.

| FPS | Frame budget |
|-----|--------------|
| 60  | 16.6 ms |
| 120 | 8.3 ms |

**Perf Monitor** (built-in, all platforms): Dev Menu → "Perf Monitor". Shows UI FPS, JS FPS, RAM live.

**Flashlight** (Android only, CI-friendly): install from a verified release channel (do not pipe install scripts to shell), then:

```bash
flashlight measure                          # interactive
flashlight measure --output results.json    # CI export
flashlight compare baseline.json current.json
```

Outputs a 0–100 score, FPS graph, CPU, RAM. Useful for FlatList → FlashList style A/B comparisons.

| Symptom | Diagnosis |
|---------|-----------|
| UI FPS drops, JS fine | Native rendering — too many views, heavy layouts, native animations |
| JS FPS drops, UI fine | JS computation blocking — re-renders, heavy work on JS thread |
| Both drop | Start with JS profiling |

**Always disable JS dev mode** before measuring: Dev Menu → Settings → JS Dev Mode → OFF (Android), or `expo start --no-dev --minify` (Expo).

## Xcode Instruments

Profile builds: **Product → Profile** (`Cmd+I`).

| Template | Purpose |
|----------|---------|
| Time Profiler | CPU hotspots, flame graph |
| Leaks | Memory leaks (red markers) |
| Allocations | All allocations over time |
| Hangs | UI thread blocks |
| Zombies | Messages to dealloc'd objects |

**Filters that matter:** Hide System Libraries, Invert Call Tree (bottom-up), filter by thread (Main, JS, native modules).

**Quick check** without Instruments: Debug Navigator → Memory/CPU graphs while running. CPU can exceed 100% (multi-core).

## Android Studio Profiler

**View → Tool Windows → Profiler** → "Find CPU Hotspots" or "Track Memory Consumption".

Views: Top-Down, Bottom-Up, Flame Chart. Filter by thread or keyword (`hermes`, `mqt_v_js`).

For advanced trace analysis, export to **Perfetto** ([ui.perfetto.dev](https://ui.perfetto.dev/)) — cross-process timeline, custom events, more visualizations than Studio.

## Bundle Analysis

```bash
# What's in your JS bundle
npx react-native bundle --platform ios --dev false \
  --entry-file index.js --bundle-output bundle.js \
  --sourcemap-output bundle.map
npx source-map-explorer bundle.js bundle.map
```

For app binary size (iOS/Android), use **Emerge Tools xray** to see which modules dominate the binary — surfaces oversized native dependencies, duplicated assets, and unstripped symbols.

See `react-native-performance/references/best-practices-bundle.md` for reduction tactics.

## Bottom Sheet Profiling

`@gorhom/bottom-sheet` gestures often surface in the React profiler as cascading commits during drag. The fix is to keep gesture-driven state on the UI thread via SharedValues — *not* React state.

```jsx
const animatedIndex = useSharedValue(0);

const overlayStyle = useAnimatedStyle(() => ({
  opacity: withTiming(animatedIndex.value > 0 ? 0.5 : 0),
}));

<BottomSheet animatedIndex={animatedIndex}>
  <ExpensiveContent />
</BottomSheet>
<Animated.View style={[styles.overlay, overlayStyle]} />
```

Use `animatedIndex` / `animatedPosition` for continuous tracking. `onChange` fires on snap completion only — don't use it for continuous logic. `onAnimate` can fire repeatedly during interaction; treat it as expensive.

For scroll inside the sheet, prefer `useAnimatedScrollHandler` over `onScroll` so work stays on the UI thread. Always use `BottomSheetScrollView` / `BottomSheetFlatList` (not RN built-ins) so gestures coordinate. If your app is on Fabric and doesn't need deep JS customization, consider `@lodev09/react-native-true-sheet` — fully native, zero JS plumbing.

## Pro Tips

1. Profile on **low-end devices** — issues are obvious there.
2. Use **release builds** with dev mode OFF.
3. Run measurements **multiple times** — FPS varies.
4. **Export traces** to compare before/after.
5. Filter by **thread** to focus on relevant work.
6. Expo Go cannot profile native code — use a Dev Client or `expo prebuild` first.

## Key Points

- Always profile a release build on a real low-end device; dev mode and simulators hide the real cost.
- Drive the audited flow during the recording — idle/startup profiles are not useful for interaction work.
- "Why did this render?" in DevTools is the actionable signal — fix the named prop/hook, not random memoization.
- UI FPS and JS FPS are independent; the dropping thread tells you whether the problem is native or JS.
- Flashlight is the only good CI-friendly FPS benchmark (Android only); export JSON and compare across builds.
- For bottom sheets and other gesture-driven UI, move state to SharedValues so the JS profiler stays quiet during interaction.

