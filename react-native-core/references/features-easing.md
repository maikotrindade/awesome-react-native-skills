---
name: react-native-easing
description: Easing functions for animations. Use when creating custom animation curves, spring animations, or fine-tuning animation timing.
---

# Easing Functions

The `Easing` module provides easing functions for `Animated.timing()` to create natural, physically believable motion.

## Predefined Animations

### back()

Object goes slightly back before moving forward.

```tsx
import { Easing } from 'react-native';

Animated.timing(value, {
  toValue: 1,
  easing: Easing.back(1.5), // Overshoot amount
  useNativeDriver: true,
}).start();
```

### bounce()

Bouncing animation effect.

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.bounce,
  useNativeDriver: true,
}).start();
```

### ease()

Basic inertial animation (default for timing).

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.ease,
  useNativeDriver: true,
}).start();
```

### elastic()

Spring-like elastic animation.

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.elastic(2), // Bounciness
  useNativeDriver: true,
}).start();
```

## Standard Functions

### linear()

Constant speed (no easing).

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.linear,
  useNativeDriver: true,
}).start();
```

### quad()

Quadratic easing (power of 2).

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.quad,
  useNativeDriver: true,
}).start();
```

### cubic()

Cubic easing (power of 3).

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.cubic,
  useNativeDriver: true,
}).start();
```

### poly()

Polynomial easing for higher powers.

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.poly(4), // Power of 4
  useNativeDriver: true,
}).start();
```

## Additional Functions

### bezier()

Cubic Bezier curve for custom easing.

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.bezier(0.25, 0.1, 0.25, 1), // x1, y1, x2, y2
  useNativeDriver: true,
}).start();
```

### circle()

Circular easing function.

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.circle,
  useNativeDriver: true,
}).start();
```

### sin()

Sinusoidal easing function.

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.sin,
  useNativeDriver: true,
}).start();
```

### exp()

Exponential easing function.

```tsx
Animated.timing(value, {
  toValue: 1,
  easing: Easing.exp,
  useNativeDriver: true,
}).start();
```

## Easing Modifiers

### in()

Runs easing function forwards (accelerate).

```tsx
Easing.in(Easing.quad)  // Accelerate with quadratic curve
```

### out()

Runs easing function backwards (decelerate).

```tsx
Easing.out(Easing.quad)  // Decelerate with quadratic curve
```

### inOut()

Makes easing function symmetrical (accelerate then decelerate).

```tsx
Easing.inOut(Easing.quad)  // Accelerate then decelerate
```

## Common Easing Combinations

### Default Timing

```tsx
Easing.inOut(Easing.ease)  // Default for Animated.timing()
```

### Smooth Entry

```tsx
Easing.out(Easing.cubic)  // Smooth deceleration
```

### Quick Exit

```tsx
Easing.in(Easing.quad)  // Quick acceleration
```

### Natural Motion

```tsx
Easing.bezier(0.4, 0.0, 0.2, 1)  // Material Design standard
```

## Usage Examples

### Fade In

```tsx
Animated.timing(opacity, {
  toValue: 1,
  duration: 300,
  easing: Easing.out(Easing.quad),
  useNativeDriver: true,
}).start();
```

### Slide In

```tsx
Animated.timing(translateX, {
  toValue: 0,
  duration: 400,
  easing: Easing.out(Easing.cubic),
  useNativeDriver: true,
}).start();
```

### Bounce Effect

```tsx
Animated.timing(scale, {
  toValue: 1,
  duration: 600,
  easing: Easing.bounce,
  useNativeDriver: true,
}).start();
```

### Elastic Spring

```tsx
Animated.timing(position, {
  toValue: 0,
  duration: 800,
  easing: Easing.elastic(2),
  useNativeDriver: true,
}).start();
```

### Custom Bezier

```tsx
// Ease-in-out cubic (smooth)
Animated.timing(value, {
  toValue: 1,
  duration: 300,
  easing: Easing.bezier(0.4, 0.0, 0.2, 1),
  useNativeDriver: true,
}).start();
```

## Best Practices

1. **Use appropriate easing** - Match animation to user action
2. **Test different curves** - Visualize at https://easings.net/
3. **Combine with duration** - Longer duration needs gentler easing
4. **Use inOut for transitions** - Most natural for state changes
5. **Use out for entrances** - Deceleration feels natural
6. **Use in for exits** - Acceleration feels quick
7. **Avoid linear** - Unless specifically needed (progress bars)

## Common Patterns

### Material Design Easing

```tsx
const materialEasing = {
  standard: Easing.bezier(0.4, 0.0, 0.2, 1),
  decelerate: Easing.bezier(0.0, 0.0, 0.2, 1),
  accelerate: Easing.bezier(0.4, 0.0, 1, 1),
  sharp: Easing.bezier(0.4, 0.0, 0.6, 1),
};
```

### iOS Easing

```tsx
const iosEasing = {
  easeInOut: Easing.inOut(Easing.ease),
  easeIn: Easing.in(Easing.ease),
  easeOut: Easing.out(Easing.ease),
};
```

### Custom Easing Presets

```tsx
const customEasing = {
  smooth: Easing.inOut(Easing.cubic),
  quick: Easing.out(Easing.quad),
  bouncy: Easing.bounce,
  elastic: Easing.elastic(1.5),
};
```

<!--
Source references:
- https://reactnative.dev/docs/easing
- https://easings.net/
-->
