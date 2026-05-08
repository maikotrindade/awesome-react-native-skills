---
name: react-native-animations
description: Built-in Animated API for creating animations. Use for simple opacity/transform animations. For complex or gesture-driven animations, prefer Reanimated v3 (see react-native-ecosystem skill).
---

# Animations

React Native ships the `Animated` API for declarative, performant animations. It is the right choice for simple, self-contained animations (fade-in, slide, scale on press). For complex, gesture-driven, scroll-driven, or layout animations, use **React Native Reanimated v3** (covered in the `react-native-ecosystem` skill).

## Built-in vs Reanimated

| Criterion | Built-in `Animated` | Reanimated v3 |
|-----------|--------------------|--------------------|
| Installation | Zero — built into RN | `npm install react-native-reanimated` |
| Thread | JS thread (with `useNativeDriver` opt-in) | Always on UI thread |
| Gesture integration | Via `PanResponder` (verbose) | `GestureDetector` (clean) |
| Scroll-driven | Limited | First-class |
| Layout animations | No | Yes (`FadeIn`, `SlideInRight`, `Layout`) |
| Worklets | No | Yes |

## Basic Animation

Create animated values and drive style properties.

```tsx
import { useRef, useEffect } from 'react';
import { Animated, View } from 'react-native';

const FadeInView = () => {
  const fadeAnim = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 1000,
      useNativeDriver: true, // required — runs on UI thread; only works with transform/opacity
    }).start();
  }, []);

  return (
    <Animated.View style={{ opacity: fadeAnim }}>
      {/* Content */}
    </Animated.View>
  );
};
```

## Animated Values

### Animated.Value

Single numeric value for animations.

```tsx
const opacity = useRef(new Animated.Value(0)).current;
const scale = useRef(new Animated.Value(1)).current;

<Animated.View style={{ opacity, transform: [{ scale }] }} />
```

### Animated.ValueXY

2D vector for pan gestures and 2D animations.

```tsx
const position = useRef(new Animated.ValueXY({ x: 0, y: 0 })).current;

<Animated.View style={{ transform: position.getTranslateTransform() }} />
```

## Animation Types

### timing() — duration-based

```tsx
Animated.timing(value, {
  toValue: 1,
  duration: 300,
  easing: Easing.out(Easing.quad),
  useNativeDriver: true,
}).start();
```

### spring() — physics-based

```tsx
Animated.spring(value, {
  toValue: 1,
  tension: 40,
  friction: 7,
  useNativeDriver: true,
}).start();
```

### decay() — velocity-based (coast to stop)

```tsx
Animated.decay(value, {
  velocity: 0.5,
  deceleration: 0.997,
  useNativeDriver: true,
}).start();
```

## Composition

### sequence — run one after another

```tsx
Animated.sequence([
  Animated.timing(opacity, { toValue: 1, duration: 300, useNativeDriver: true }),
  Animated.timing(scale, { toValue: 1.2, duration: 200, useNativeDriver: true }),
  Animated.timing(scale, { toValue: 1, duration: 150, useNativeDriver: true }),
]).start();
```

### parallel — run simultaneously

```tsx
Animated.parallel([
  Animated.timing(opacity, { toValue: 1, duration: 300, useNativeDriver: true }),
  Animated.timing(translateY, { toValue: 0, duration: 300, useNativeDriver: true }),
]).start();
```

### stagger — parallel with delays

```tsx
Animated.stagger(
  100, // delay between each
  items.map((_, i) =>
    Animated.timing(animations[i], { toValue: 1, duration: 300, useNativeDriver: true })
  )
).start();
```

### loop — repeat indefinitely

```tsx
Animated.loop(
  Animated.timing(spin, {
    toValue: 1,
    duration: 1000,
    easing: Easing.linear,
    useNativeDriver: true,
  })
).start();
```

## Interpolation

Map animated value to different output ranges.

```tsx
const rotation = fadeAnim.interpolate({
  inputRange: [0, 1],
  outputRange: ['0deg', '360deg'],
  extrapolate: 'clamp',
});

<Animated.View style={{ transform: [{ rotate: rotation }] }} />
```

```tsx
// Color interpolation (cannot use useNativeDriver)
const color = progress.interpolate({
  inputRange: [0, 1],
  outputRange: ['rgb(255,0,0)', 'rgb(0,255,0)'],
});
```

## useNativeDriver Limitation

`useNativeDriver: true` only works with properties that can be run entirely on the native/UI thread:

**Supported**: `opacity`, `transform` (translateX/Y, scale, rotate, skew)

**Not supported**: `backgroundColor`, `height`, `width`, `padding`, `margin`, `borderRadius` — use `useNativeDriver: false` for these, or Reanimated v3 which supports all style properties on the UI thread.

## PanResponder (gesture-based animation)

```tsx
const pan = useRef(new Animated.ValueXY()).current;

const panResponder = useRef(
  PanResponder.create({
    onStartShouldSetPanResponder: () => true,
    onPanResponderMove: Animated.event(
      [null, { dx: pan.x, dy: pan.y }],
      { useNativeDriver: false } // PanResponder + ValueXY requires false
    ),
    onPanResponderRelease: () => {
      Animated.spring(pan, {
        toValue: { x: 0, y: 0 },
        useNativeDriver: false,
      }).start();
    },
  })
).current;

return (
  <Animated.View
    {...panResponder.panHandlers}
    style={{ transform: pan.getTranslateTransform() }}
  />
);
```

> For drag gestures in new code, prefer `react-native-gesture-handler` + `react-native-reanimated` — the PanResponder API is verbose and runs on the JS thread.

## LayoutAnimation

Automatically animate layout changes when state updates.

```tsx
import { LayoutAnimation, Platform, UIManager } from 'react-native';

// Required on Android
if (Platform.OS === 'android') {
  UIManager.setLayoutAnimationEnabledExperimental(true);
}

const expand = () => {
  LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut);
  setExpanded(true); // layout change will animate automatically
};
```

Presets: `easeInEaseOut`, `linear`, `spring`.

> **Note**: `LayoutAnimation` is incompatible with the New Architecture on Android. Use Reanimated v3 `Layout` transitions instead.
