# Features — React Native Reanimated v3

Reanimated v3 runs animations on the UI thread via worklets, eliminating JS bridge overhead. Always prefer it over the legacy `Animated` API.

## Installation

```bash
npm install react-native-reanimated
```

Add to `babel.config.js`:
```js
module.exports = {
  plugins: ['react-native-reanimated/plugin'], // must be last plugin
};
```

## Core Primitives

| Primitive | Purpose |
|-----------|---------|
| `useSharedValue` | Mutable value that lives on UI thread |
| `useAnimatedStyle` | Derive animated styles from shared values |
| `useDerivedValue` | Compute a new shared value from others |
| `useAnimatedProps` | Animate non-style props (SVG, etc.) |
| `runOnJS` | Call a JS-thread function from a worklet |
| `runOnUI` | Schedule a worklet on the UI thread |

## Basic Animation

```tsx
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withSpring,
  Easing,
} from 'react-native-reanimated';

function FadeInBox() {
  const opacity = useSharedValue(0);

  useEffect(() => {
    opacity.value = withTiming(1, { duration: 400, easing: Easing.out(Easing.quad) });
  }, []);

  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
    transform: [{ scale: opacity.value }],
  }));

  return <Animated.View style={[styles.box, animatedStyle]} />;
}
```

## Animation Drivers

```ts
// withTiming — linear/eased interpolation
opacity.value = withTiming(1, { duration: 300 });

// withSpring — physics-based spring
scale.value = withSpring(1.2, { damping: 10, stiffness: 100, mass: 1 });

// withDelay — delay before animation starts
opacity.value = withDelay(500, withTiming(1));

// withRepeat — repeat N times (-1 = infinite)
scale.value = withRepeat(withTiming(1.1, { duration: 600 }), -1, true); // true = reverse

// withSequence — run animations one after another
offset.value = withSequence(
  withTiming(-10, { duration: 50 }),
  withRepeat(withTiming(10, { duration: 100 }), 4, true),
  withTiming(0, { duration: 50 })
);
```

## useDerivedValue

```ts
const progress = useSharedValue(0); // 0 → 1

// Derived value recomputes whenever `progress` changes
const color = useDerivedValue(() =>
  interpolateColor(progress.value, [0, 1], ['#ff0000', '#00ff00'])
);

const animatedStyle = useAnimatedStyle(() => ({
  backgroundColor: color.value,
}));
```

## Interpolate

```ts
import { interpolate, Extrapolation } from 'react-native-reanimated';

const animatedStyle = useAnimatedStyle(() => {
  const scale = interpolate(
    scrollY.value,          // input value
    [0, 100, 200],          // input range
    [1, 0.8, 0],            // output range
    Extrapolation.CLAMP     // clamp outside range
  );
  return { transform: [{ scale }] };
});
```

## Scroll-Driven Animations

```tsx
import Animated, { useAnimatedScrollHandler, useSharedValue } from 'react-native-reanimated';

function AnimatedHeader() {
  const scrollY = useSharedValue(0);

  const scrollHandler = useAnimatedScrollHandler({
    onScroll: (event) => {
      scrollY.value = event.contentOffset.y;
    },
  });

  const headerStyle = useAnimatedStyle(() => ({
    opacity: interpolate(scrollY.value, [0, 80], [1, 0], Extrapolation.CLAMP),
    transform: [{ translateY: interpolate(scrollY.value, [0, 80], [0, -40], Extrapolation.CLAMP) }],
  }));

  return (
    <>
      <Animated.View style={[styles.header, headerStyle]} />
      <Animated.ScrollView onScroll={scrollHandler} scrollEventThrottle={16}>
        {/* content */}
      </Animated.ScrollView>
    </>
  );
}
```

## Layout Animations

```tsx
import Animated, { FadeIn, FadeOut, SlideInRight, Layout } from 'react-native-reanimated';

// Entering/exiting animations
<Animated.View entering={FadeIn.duration(300)} exiting={FadeOut.duration(200)}>
  <Text>Appears and disappears with fade</Text>
</Animated.View>

// Layout transition — animates when position/size changes
<Animated.View layout={Layout.springify()}>
  {/* Re-renders with spring animation when layout changes */}
</Animated.View>

// Conditional rendering with animation
{isVisible && (
  <Animated.View entering={SlideInRight} exiting={FadeOut}>
    <Card />
  </Animated.View>
)}
```

## Keyframe Animations

```ts
import { Keyframe } from 'react-native-reanimated';

const enteringAnimation = new Keyframe({
  0: { opacity: 0, transform: [{ scale: 0.5 }] },
  60: { opacity: 1, transform: [{ scale: 1.1 }] },
  100: { opacity: 1, transform: [{ scale: 1 }] },
}).duration(400);

<Animated.View entering={enteringAnimation}>
```

## runOnJS — Calling JS from Worklets

```ts
// Worklets run on UI thread. Use runOnJS to call JS-thread functions.
import { runOnJS } from 'react-native-reanimated';

const onGestureEnd = useCallback((completed: boolean) => {
  if (completed) navigation.goBack();
}, []);

const gestureHandler = useAnimatedGestureHandler({
  onEnd: (_, ctx) => {
    if (ctx.translateX > 100) {
      runOnJS(onGestureEnd)(true); // safe cross-thread call
    }
  },
});
```

## useAnimatedRef + measure

```tsx
import Animated, { useAnimatedRef, measure, runOnUI } from 'react-native-reanimated';

function MeasureExample() {
  const aref = useAnimatedRef<Animated.View>();

  const handlePress = () => {
    runOnUI(() => {
      const layout = measure(aref);
      if (layout) {
        console.log(layout.width, layout.height, layout.pageX, layout.pageY);
      }
    })();
  };

  return <Animated.View ref={aref} onPress={handlePress} />;
}
```

## Shared Element Transitions (React Navigation)

```tsx
import Animated from 'react-native-reanimated';

// In list screen
<Animated.Image
  source={{ uri: item.imageUrl }}
  sharedTransitionTag={`image-${item.id}`}
  style={styles.thumbnail}
/>

// In detail screen
<Animated.Image
  source={{ uri: item.imageUrl }}
  sharedTransitionTag={`image-${item.id}`}
  style={styles.fullImage}
/>
```

## Performance Notes

- Worklets (functions inside `useAnimatedStyle`, `useAnimatedScrollHandler`, etc.) run entirely on the UI thread — no JS bridge needed
- Never reference JS-side variables inside worklets that can't be serialized; use `useDerivedValue` to bridge
- `runOnJS` incurs a cross-thread hop — avoid it in tight animation loops
- Prefer `withSpring` over rapid `withTiming` chains for more natural feel
- Layout animations on long lists can be expensive — scope to visible items
