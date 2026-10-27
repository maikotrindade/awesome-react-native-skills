# Features — React Native Reanimated 4

Reanimated 4 runs animations on the UI thread. It has two APIs that can be mixed freely:

- **CSS animations and transitions** — declarative, style-based, no worklets. Use for state-driven UI (toggles, enter/hover effects, loaders).
- **Shared values and worklets** — imperative, frame-by-frame. Use for gesture-driven, scroll-driven, or physics-based motion.

> Reanimated 4.7 requires React Native **0.86–0.88**, the **New Architecture**, and `react-native-worklets` **0.13.x**. Projects still on the old architecture must stay on Reanimated 3.x. Prefer Reanimated over the built-in `Animated` API for anything non-trivial.

## Installation

```bash
npx expo install react-native-reanimated react-native-worklets   # Expo
npm install react-native-reanimated react-native-worklets         # bare RN
```

The Babel plugin moved to the worklets package. In bare projects, add it as the **last** plugin:

```js
// babel.config.js
module.exports = {
  presets: ['module:@react-native/babel-preset'],
  plugins: ['react-native-worklets/plugin'], // was 'react-native-reanimated/plugin' in v3
};
```

`babel-preset-expo` adds the plugin automatically, so Expo projects need no Babel changes. After installing, rebuild the native app and clear the Metro cache (`npx expo start -c` / `npx react-native start --reset-cache`).

## Choosing an API

| Situation | Use |
|-----------|-----|
| Style changes when React state changes | CSS transition (`transitionProperty`) |
| Looping or keyframed motion (spinner, pulse, shimmer) | CSS animation (`animationName`) |
| Element mounts / unmounts / reorders | Layout animations (`entering`, `exiting`, `layout`) |
| Follows a finger or scroll position | Shared values + `useAnimatedStyle` |
| Physics (springs with velocity, decay) | `withSpring` / `withDecay` |

## CSS Transitions

Animate between style values whenever the style changes on re-render:

```tsx
import Animated from 'react-native-reanimated';

function ExpandableCard({ expanded }: { expanded: boolean }) {
  return (
    <Animated.View
      style={{
        height: expanded ? 240 : 80,
        backgroundColor: expanded ? '#1e293b' : '#334155',
        transitionProperty: ['height', 'backgroundColor'],
        transitionDuration: 250,
        transitionTimingFunction: 'ease-out',
      }}
    />
  );
}
```

- `transitionProperty` accepts a single property, an array, or `'all'` (avoid `'all'` on large style objects).
- `transitionDuration` / `transitionDelay` accept numbers (ms) or strings (`'250ms'`, `'0.25s'`).
- Timing functions: `'ease'`, `'linear'`, `'ease-in-out'`, or `cubicBezier()`, `steps()`, `linear()` imported from `react-native-reanimated`.

## CSS Animations

Define keyframes once with `css.keyframes` and reference them from a style:

```tsx
import Animated, { css } from 'react-native-reanimated';

const pulse = css.keyframes({
  from: { opacity: 1, transform: [{ scale: 1 }] },
  '50%': { opacity: 0.6, transform: [{ scale: 0.96 }] },
  to: { opacity: 1, transform: [{ scale: 1 }] },
});

const styles = css.create({
  skeleton: {
    animationName: pulse,
    animationDuration: '1.2s',
    animationIterationCount: 'infinite',
    animationTimingFunction: 'ease-in-out',
  },
});

export function SkeletonRow() {
  return <Animated.View style={[{ height: 16, borderRadius: 8, backgroundColor: '#e5e7eb' }, styles.skeleton]} />;
}
```

- Toggle `animationPlayState: 'paused' | 'running'` to pause without unmounting.
- `animationFillMode`, `animationDirection`, and `animationDelay` follow the web CSS semantics.
- Lifecycle callbacks are style keys next to the animation config: `onCSSAnimationStart`, `onCSSAnimationEnd`, `onCSSAnimationIteration`, and for transitions `onCSSTransitionRun`, `onCSSTransitionStart`, `onCSSTransitionEnd`.
- Define keyframes and `css.create` styles at module scope so they are not recreated on every render.

## Shared Values

| Primitive | Purpose |
|-----------|---------|
| `useSharedValue(initial)` | Mutable value readable on both JS and UI threads |
| `useAnimatedStyle(() => style)` | Style recomputed on the UI thread when shared values change |
| `useDerivedValue(() => value)` | Read-only value computed from other shared values |
| `useAnimatedProps(() => props)` | Animate non-style props (SVG attributes, `TextInput` text) |
| `useAnimatedReaction(prepare, react)` | Run a side effect when a derived value changes |
| `useAnimatedRef()` + `measure()` | Read layout synchronously on the UI thread |

```tsx
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';

function LikeButton() {
  const scale = useSharedValue(1);
  const style = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }));

  return (
    <Pressable
      onPressIn={() => (scale.value = withSpring(0.9))}
      onPressOut={() => (scale.value = withSpring(1))}
    >
      <Animated.View style={style}>{/* icon */}</Animated.View>
    </Pressable>
  );
}
```

With the React Compiler enabled, use `sv.get()` / `sv.set(v)` instead of `.value` — the compiler cannot track property mutation on shared values.

## Animation Functions

```ts
x.value = withTiming(100, { duration: 300, easing: Easing.out(Easing.cubic) });
x.value = withSpring(100);                                  // default spring
x.value = withSpring(100, { duration: 400, dampingRatio: 0.8 }); // duration-based spring
x.value = withDecay({ velocity: e.velocityX, clamp: [0, 300] });
x.value = withDelay(200, withTiming(1));
x.value = withRepeat(withTiming(1.1, { duration: 600 }), -1, true); // -1 = infinite, true = reverse
x.value = withSequence(withTiming(-8), withTiming(8), withTiming(0));
x.value = withClamp({ min: 0, max: 200 }, withSpring(250));        // clamp a spring's overshoot
cancelAnimation(x);
```

Reanimated 4 changed the default spring to a softer, duration-based config. Ready-made presets are exported: `GentleSpringConfig`, `SnappySpringConfig`, `WigglySpringConfig`. Use `Reanimated3DefaultSpringConfig` to reproduce the v3 feel during a migration.

## Interpolation

```ts
import { interpolate, interpolateColor, Extrapolation } from 'react-native-reanimated';

const style = useAnimatedStyle(() => ({
  opacity: interpolate(scrollY.value, [0, 80], [1, 0], Extrapolation.CLAMP),
  backgroundColor: interpolateColor(progress.value, [0, 1], ['#ef4444', '#22c55e']),
}));
```

## Scroll-Driven Animations

```tsx
import Animated, { useAnimatedRef, useScrollOffset, useAnimatedStyle, interpolate, Extrapolation } from 'react-native-reanimated';

function CollapsingHeader() {
  const scrollRef = useAnimatedRef<Animated.ScrollView>();
  const offset = useScrollOffset(scrollRef);

  const headerStyle = useAnimatedStyle(() => ({
    transform: [{ translateY: interpolate(offset.value, [0, 120], [0, -60], Extrapolation.CLAMP) }],
  }));

  return (
    <>
      <Animated.View style={[styles.header, headerStyle]} />
      <Animated.ScrollView ref={scrollRef}>{/* content */}</Animated.ScrollView>
    </>
  );
}
```

`useScrollOffset` is the current name; `useScrollViewOffset` remains as an alias. Use `useAnimatedScrollHandler` when you need `onBeginDrag` / `onMomentumEnd` callbacks.

## Layout Animations

```tsx
import Animated, { FadeIn, FadeOut, LinearTransition } from 'react-native-reanimated';

{items.map((item) => (
  <Animated.View
    key={item.id}
    entering={FadeIn.duration(200)}
    exiting={FadeOut.duration(150)}
    layout={LinearTransition.springify()}
  >
    <Row item={item} />
  </Animated.View>
))}
```

- `Layout` is a deprecated alias — use `LinearTransition` (or `SequencedTransition`, `FadingTransition`, `CurvedTransition`).
- Custom entrances: `new Keyframe({ 0: {...}, 100: {...} }).duration(400)`.
- Wrap a subtree in `<LayoutAnimationConfig skipEntering>` to suppress entering animations on the first render (e.g., a list that is already populated).
- `Animated.FlatList` accepts `itemLayoutAnimation` for reorder/remove animations.

## Calling JS from the UI Thread

Worklet-thread helpers now live in `react-native-worklets`. `runOnJS` and `runOnUI` still work but are deprecated:

```ts
import { scheduleOnRN, scheduleOnUI } from 'react-native-worklets';

// UI → JS: arguments are passed directly, not curried
scheduleOnRN(onDismiss, itemId);          // v3: runOnJS(onDismiss)(itemId)

// JS → UI
scheduleOnUI(() => {
  'worklet';
  const layout = measure(aref);
});                                        // v3: runOnUI(() => {...})()
```

See [features-worklets](features-worklets.md) for threads, runtimes, and shared memory.

## Shared Element Transitions

Shared element transitions are **off by default** in Reanimated 4. Enable the static feature flag in the app's `package.json`, then rebuild:

```json
{
  "reanimated": {
    "staticFeatureFlags": { "ENABLE_SHARED_ELEMENT_TRANSITIONS": true }
  }
}
```

```tsx
<Animated.Image sharedTransitionTag={`photo-${id}`} source={{ uri }} style={styles.thumb} />
```

Use matching `sharedTransitionTag`s on both screens of a native stack. `SharedTransitionBoundary` (`isActive`) scopes which subtree participates. Treat the feature as experimental and test on both platforms.

## Accessibility

Respect the OS "reduce motion" setting:

```tsx
import { ReducedMotionConfig, ReduceMotion, useReducedMotion } from 'react-native-reanimated';

<ReducedMotionConfig mode={ReduceMotion.System} />   // app-wide default
const reduceMotion = useReducedMotion();              // per-component check
x.value = withTiming(1, { reduceMotion: ReduceMotion.System });
```

## Migrating from Reanimated 3

| v3 | v4 |
|----|----|
| `react-native-reanimated/plugin` | `react-native-worklets/plugin` |
| `runOnJS(fn)(arg)` | `scheduleOnRN(fn, arg)` |
| `runOnUI(fn)()` | `scheduleOnUI(fn)` |
| `useAnimatedGestureHandler` | Removed — use Gesture Handler callbacks ([features-gesture-handler](features-gesture-handler.md)) |
| `useScrollViewOffset` | `useScrollOffset` (old name kept as alias) |
| `Layout` transition | `LinearTransition` |
| `useWorkletCallback` | Removed — use `useCallback` with a `'worklet'` directive |
| Default spring (stiffness-based) | Duration-based; use `Reanimated3DefaultSpringConfig` to keep old feel |
| Old Architecture supported | New Architecture only |

## Performance Notes

- CSS animations and transitions avoid per-frame JS entirely — prefer them when motion is not tied to input.
- Animate `transform` and `opacity` where possible; animating `width`/`height`/`top` triggers layout on every frame.
- Keep worklets small and avoid capturing large objects — everything captured is copied to the UI runtime.
- Avoid `scheduleOnRN` inside `onUpdate`-style callbacks that fire every frame.
- Use `PerformanceMonitor` (exported from Reanimated) during development to watch UI-thread FPS.
- Test with `setUpTests()` and `withReanimatedTimer` / `advanceAnimationByTime` from Reanimated's Jest utilities.
