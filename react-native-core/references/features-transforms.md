---
name: react-native-transforms
description: Transform style properties for 2D and 3D transformations. Use when rotating, scaling, translating, or skewing components.
---

# Transforms

Transform properties modify component appearance and position using 2D or 3D transformations. Transforms don't affect layout - transformed components may overlap nearby elements.

## 2D Transforms

### translateX / translateY

Move component along X or Y axis.

```tsx
<View style={{
  transform: [
    { translateX: 50 },  // Move 50px right
    { translateY: -20 }, // Move 20px up
  ],
}} />
```

### scale / scaleX / scaleY

Scale component uniformly or along specific axis.

```tsx
<View style={{
  transform: [
    { scale: 2 },        // Scale 2x uniformly
    { scaleX: 1.5 },    // Scale 1.5x horizontally
    { scaleY: 0.5 },    // Scale 0.5x vertically
  ],
}} />
```

### rotate

Rotate component around Z axis.

```tsx
<View style={{
  transform: [
    { rotate: '45deg' },
    { rotate: '1.57rad' },
  ],
}} />
```

### skewX / skewY

Skew component along X or Y axis.

```tsx
<View style={{
  transform: [
    { skewX: '45deg' },
    { skewY: '30deg' },
  ],
}} />
```

## 3D Transforms

### rotateX / rotateY / rotateZ

Rotate around specific axis.

```tsx
<View style={{
  transform: [
    { rotateX: '45deg' },  // Rotate around X axis
    { rotateY: '30deg' },  // Rotate around Y axis
    { rotateZ: '60deg' },  // Rotate around Z axis
  ],
}} />
```

### perspective

Add perspective for 3D transforms.

```tsx
<View style={{
  transform: [
    { perspective: 1000 },
    { rotateY: '45deg' },
  ],
}} />
```

## Combining Transforms

Apply multiple transforms in order.

```tsx
<View style={{
  transform: [
    { translateX: 50 },
    { rotate: '45deg' },
    { scale: 1.5 },
  ],
}} />
```

**Order matters**: Transforms apply sequentially.

## Transform with Animated

Use transforms with Animated API for smooth animations.

```tsx
import { useRef, useEffect } from 'react';
import { Animated } from 'react-native';

const App = () => {
  const scale = useRef(new Animated.Value(1)).current;
  const rotate = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    Animated.parallel([
      Animated.spring(scale, {
        toValue: 1.5,
        useNativeDriver: true,
      }),
      Animated.timing(rotate, {
        toValue: 360,
        duration: 1000,
        useNativeDriver: true,
      }),
    ]).start();
  }, []);

  return (
    <Animated.View style={{
      transform: [
        { scale },
        { rotate: rotate.interpolate({
          inputRange: [0, 360],
          outputRange: ['0deg', '360deg'],
        })},
      ],
    }}>
      {/* Content */}
    </Animated.View>
  );
};
```

## Common Patterns

### Hover/Press Scale

```tsx
const [pressed, setPressed] = useState(false);

<Animated.View
  style={{
    transform: [{ scale: pressed ? 1.1 : 1 }],
  }}
  onPressIn={() => setPressed(true)}
  onPressOut={() => setPressed(false)}
/>
```

### Rotation Animation

```tsx
const rotate = useRef(new Animated.Value(0)).current;

useEffect(() => {
  Animated.loop(
    Animated.timing(rotate, {
      toValue: 360,
      duration: 2000,
      useNativeDriver: true,
    })
  ).start();
}, []);

<Animated.View style={{
  transform: [{
    rotate: rotate.interpolate({
      inputRange: [0, 360],
      outputRange: ['0deg', '360deg'],
    }),
  }],
}} />
```

### Flip Card

```tsx
const flip = useRef(new Animated.Value(0)).current;

const flipCard = () => {
  Animated.spring(flip, {
    toValue: flip._value === 0 ? 180 : 0,
    useNativeDriver: true,
  }).start();
};

<Animated.View style={{
  transform: [{
    rotateY: flip.interpolate({
      inputRange: [0, 180],
      outputRange: ['0deg', '180deg'],
    }),
  }],
}}>
  {/* Card content */}
</Animated.View>
```

## Best Practices

1. **Use transforms for animations** - More performant than layout properties
2. **Combine with Animated API** - Smooth, native-driven animations
3. **Handle overlap** - Add margin/padding to prevent overlap
4. **Use native driver** - Transforms support `useNativeDriver: true`
5. **Order matters** - Transforms apply sequentially
6. **Test on device** - 3D transforms may behave differently

## Performance Notes

- **Transforms don't trigger layout** - More performant than changing width/height
- **Use for animations** - Prefer transforms over layout properties
- **Native driver support** - All transforms work with `useNativeDriver: true`
- **GPU accelerated** - Transforms are hardware accelerated

<!--
Source references:
- https://reactnative.dev/docs/transforms
-->
