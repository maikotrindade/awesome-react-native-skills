---
name: react-native-touch-handling
description: Handling touch interactions with Pressable, Touchable components, and gesture responders. Use when creating buttons, handling taps, or implementing custom touch interactions.
---

# Touch Handling

React Native provides multiple ways to handle touch interactions, from simple buttons to complex gesture recognition.

## Pressable

Modern, flexible component for handling press interactions. Recommended for new code.

```tsx
import { Pressable, Text } from 'react-native';

const App = () => {
  const [pressed, setPressed] = useState(false);

  return (
    <Pressable
      onPress={() => console.log('pressed')}
      onPressIn={() => setPressed(true)}
      onPressOut={() => setPressed(false)}
      style={({ pressed }) => [
        styles.button,
        pressed && styles.pressed
      ]}
    >
      {({ pressed }) => (
        <Text style={pressed ? styles.pressedText : styles.text}>
          Press Me
        </Text>
      )}
    </Pressable>
  );
};
```

### Pressable Props

```tsx
<Pressable
  onPress={() => {}}              // Called on press release
  onPressIn={() => {}}            // Called when press starts
  onPressOut={() => {}}           // Called when press ends
  onLongPress={() => {}}          // Called after 500ms hold
  onPressStart={() => {}}         // iOS only - press starts
  onPressEnd={() => {}}           // iOS only - press ends
  delayLongPress={500}            // Long press delay (ms)
  hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }} // Expand touch area
  pressRetentionOffset={{ top: 20, bottom: 20 }} // Keep press active when moving
  android_ripple={{ color: 'blue' }} // Android ripple effect
  disabled={false}
  style={({ pressed }) => [styles.button, pressed && styles.pressed]}
>
  {({ pressed }) => <Text>{pressed ? 'Pressed' : 'Press Me'}</Text>}
</Pressable>
```

### Pressable States

The style function receives a state object:

```tsx
<Pressable
  style={({ pressed, hovered, focused }) => [
    styles.button,
    pressed && styles.pressed,
    hovered && styles.hovered,    // Web only
    focused && styles.focused,    // Web only
  ]}
>
  {({ pressed }) => <Text>{pressed ? 'Pressed!' : 'Press Me'}</Text>}
</Pressable>
```

## Button

Simple, platform-styled button component.

```tsx
import { Button, Alert } from 'react-native';

<Button
  title="Press Me"
  onPress={() => Alert.alert('Button pressed')}
  color="#841584" // iOS only
  disabled={false}
/>
```

**Limitations:**
- Limited styling options
- Platform-specific appearance
- No custom children

## Touchable Components

Legacy touchable components (still supported, but Pressable is preferred).

### TouchableOpacity

Reduces opacity when pressed.

```tsx
import { TouchableOpacity, Text } from 'react-native';

<TouchableOpacity
  onPress={() => {}}
  activeOpacity={0.7} // Opacity when pressed (0-1)
  disabled={false}
>
  <Text>Press Me</Text>
</TouchableOpacity>
```

### TouchableHighlight

Darkens background when pressed.

```tsx
import { TouchableHighlight, Text } from 'react-native';

<TouchableHighlight
  onPress={() => {}}
  underlayColor="#DDDDDD" // Background color when pressed
  activeOpacity={0.7}
>
  <Text>Press Me</Text>
</TouchableHighlight>
```

### TouchableNativeFeedback

Android ripple effect (Android only).

```tsx
import { TouchableNativeFeedback, Text, Platform } from 'react-native';

{Platform.OS === 'android' && (
  <TouchableNativeFeedback
    onPress={() => {}}
    background={TouchableNativeFeedback.SelectableBackground()}
  >
    <Text>Press Me</Text>
  </TouchableNativeFeedback>
)}
```

### TouchableWithoutFeedback

No visual feedback (use sparingly).

```tsx
import { TouchableWithoutFeedback, Text } from 'react-native';

<TouchableWithoutFeedback onPress={() => {}}>
  <Text>Press Me</Text>
</TouchableWithoutFeedback>
```

## Common Touchable Props

All touchable components share these props:

```tsx
<TouchableOpacity
  onPress={() => {}}              // Called on press
  onPressIn={() => {}}            // Called when press starts
  onPressOut={() => {}}           // Called when press ends
  onLongPress={() => {}}          // Called after delayLongPress
  delayLongPress={500}            // Long press delay (ms)
  delayPressIn={0}                // Delay before onPressIn
  delayPressOut={100}             // Delay before onPressOut
  disabled={false}
  hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}
  pressRetentionOffset={{ top: 20, bottom: 20 }}
/>
```

## PanResponder

Advanced gesture responder for complex multi-touch gestures. Reconciles multiple touches into a single gesture.

### Basic Usage

```tsx
import { useRef } from 'react';
import { PanResponder, Animated, View } from 'react-native';

const App = () => {
  const pan = useRef(new Animated.ValueXY()).current;

  const panResponder = useRef(
    PanResponder.create({
      onStartShouldSetPanResponder: () => true,
      onMoveShouldSetPanResponder: () => true,
      onPanResponderGrant: () => {
        pan.setOffset({
          x: pan.x._value,
          y: pan.y._value,
        });
      },
      onPanResponderMove: Animated.event(
        [null, { dx: pan.x, dy: pan.y }],
        { useNativeDriver: false }
      ),
      onPanResponderRelease: () => {
        pan.flattenOffset();
      },
    })
  ).current;

  return (
    <Animated.View
      style={{
        transform: [{ translateX: pan.x }, { translateY: pan.y }],
      }}
      {...panResponder.panHandlers}
    />
  );
};
```

### PanResponder Handlers

```tsx
PanResponder.create({
  // Should become responder?
  onStartShouldSetPanResponder: (evt, gestureState) => true,
  onMoveShouldSetPanResponder: (evt, gestureState) => true,
  
  // Capture phase (before bubbling)
  onStartShouldSetPanResponderCapture: (evt, gestureState) => false,
  onMoveShouldSetPanResponderCapture: (evt, gestureState) => false,
  
  // Gesture lifecycle
  onPanResponderGrant: (evt, gestureState) => {
    // Gesture started
  },
  onPanResponderMove: (evt, gestureState) => {
    // Gesture moving
  },
  onPanResponderRelease: (evt, gestureState) => {
    // Gesture ended
  },
  onPanResponderTerminate: (evt, gestureState) => {
    // Gesture cancelled
  },
  
  // Termination handling
  onPanResponderTerminationRequest: (evt, gestureState) => true,
  onShouldBlockNativeResponder: (evt, gestureState) => true,
});
```

### GestureState

Available in all handlers:

```tsx
{
  stateID: number,           // Gesture ID
  moveX: number,             // Latest X coordinate
  moveY: number,             // Latest Y coordinate
  x0: number,                // Initial X coordinate
  y0: number,                // Initial Y coordinate
  dx: number,                // Accumulated X distance
  dy: number,                // Accumulated Y distance
  vx: number,                // X velocity
  vy: number,                // Y velocity
  numberActiveTouches: number,
}
```

### Common Patterns

**Drag and Drop:**

```tsx
const panResponder = PanResponder.create({
  onMoveShouldSetPanResponder: () => true,
  onPanResponderMove: (evt, gestureState) => {
    setPosition({
      x: gestureState.dx,
      y: gestureState.dy,
    });
  },
  onPanResponderRelease: () => {
    // Snap to final position
  },
});
```

**Swipe Detection:**

```tsx
const panResponder = PanResponder.create({
  onMoveShouldSetPanResponder: () => true,
  onPanResponderRelease: (evt, gestureState) => {
    if (Math.abs(gestureState.dx) > 100) {
      // Swipe detected
      if (gestureState.dx > 0) {
        // Swipe right
      } else {
        // Swipe left
      }
    }
  },
});
```

## Gesture Responder System

For advanced gesture recognition, use PanResponder (see above).

```tsx
import { useRef } from 'react';
import { PanResponder, Animated, View } from 'react-native';

const App = () => {
  const pan = useRef(new Animated.ValueXY()).current;

  const panResponder = useRef(
    PanResponder.create({
      onStartShouldSetPanResponder: () => true,
      onMoveShouldSetPanResponder: () => true,
      onPanResponderGrant: () => {
        pan.setOffset({
          x: pan.x._value,
          y: pan.y._value,
        });
      },
      onPanResponderMove: Animated.event(
        [null, { dx: pan.x, dy: pan.y }],
        { useNativeDriver: false }
      ),
      onPanResponderRelease: () => {
        pan.flattenOffset();
      },
    })
  ).current;

  return (
    <Animated.View
      style={{
        transform: [{ translateX: pan.x }, { translateY: pan.y }],
      }}
      {...panResponder.panHandlers}
    />
  );
};
```

### PanResponder Handlers

```tsx
PanResponder.create({
  // Should this view become the responder?
  onStartShouldSetPanResponder: (evt, gestureState) => true,
  onMoveShouldSetPanResponder: (evt, gestureState) => true,

  // Grant responder (gesture started)
  onPanResponderGrant: (evt, gestureState) => {},

  // Move during gesture
  onPanResponderMove: (evt, gestureState) => {},

  // Release responder (gesture ended)
  onPanResponderRelease: (evt, gestureState) => {},

  // Terminate responder (interrupted)
  onPanResponderTerminate: (evt, gestureState) => {},
});
```

### GestureState

Available in all PanResponder handlers:

```tsx
{
  stateID: number,        // ID of gesture
  moveX: number,          // Latest screen X coordinate
  moveY: number,          // Latest screen Y coordinate
  x0: number,             // Initial screen X coordinate
  y0: number,             // Initial screen Y coordinate
  dx: number,             // Accumulated distance moved (x)
  dy: number,             // Accumulated distance moved (y)
  vx: number,             // Current velocity (x)
  vy: number,             // Current velocity (y)
  numberActiveTouches: number,
}
```

## Best Practices

1. **Use Pressable** for new code - most flexible and modern
2. **Use Button** for simple, platform-styled buttons
3. **Use TouchableOpacity** for simple opacity feedback
4. **Use PanResponder** for complex gestures (drag, pinch, etc.)
5. **Set hitSlop** to increase touch area for small buttons
6. **Handle disabled state** to prevent interactions
7. **Provide visual feedback** - users expect it
8. **Test on device** - touch behavior differs from simulator

## Common Patterns

### Custom Button Component

```tsx
const CustomButton = ({ onPress, title, disabled }) => (
  <Pressable
    onPress={onPress}
    disabled={disabled}
    style={({ pressed }) => [
      styles.button,
      pressed && styles.pressed,
      disabled && styles.disabled,
    ]}
  >
    <Text style={styles.text}>{title}</Text>
  </Pressable>
);
```

### Long Press Handler

```tsx
<Pressable
  onLongPress={() => {
    Alert.alert('Long press detected');
  }}
  delayLongPress={500}
>
  <Text>Long press me</Text>
</Pressable>
```

### Drag and Drop

```tsx
const panResponder = PanResponder.create({
  onMoveShouldSetPanResponder: () => true,
  onPanResponderMove: (evt, gestureState) => {
    // Update position based on gestureState.dx, gestureState.dy
  },
  onPanResponderRelease: () => {
    // Snap to final position or animate
  },
});
```

<!--
Source references:
- https://reactnative.dev/docs/pressable
- https://reactnative.dev/docs/button
- https://reactnative.dev/docs/handling-touches
- https://reactnative.dev/docs/gesture-responder-system
- https://reactnative.dev/docs/touchableopacity
- https://reactnative.dev/docs/touchablehighlight
- https://reactnative.dev/docs/touchablenativefeedback
- https://reactnative.dev/docs/touchablewithoutfeedback
-->
