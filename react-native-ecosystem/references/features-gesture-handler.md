# Features — React Native Gesture Handler v2

Gesture Handler v2 uses the native gesture recognizer system on each platform. The compositional `GestureDetector` + `Gesture.*` API (v2) replaces the old handler components.

## Installation

```bash
npm install react-native-gesture-handler
```

Wrap your root component:
```tsx
// App.tsx — must be the outermost wrapper
import { GestureHandlerRootView } from 'react-native-gesture-handler';

export default function App() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      {/* rest of app */}
    </GestureHandlerRootView>
  );
}
```

## Core API

| Class | Gesture |
|-------|---------|
| `Gesture.Tap()` | Single/multi tap |
| `Gesture.LongPress()` | Long press |
| `Gesture.Pan()` | Drag / swipe |
| `Gesture.Pinch()` | Zoom with two fingers |
| `Gesture.Rotation()` | Rotate with two fingers |
| `Gesture.Fling()` | Quick fling in a direction |
| `Gesture.Native()` | Defer to native scroll/gesture |

## GestureDetector

```tsx
import { GestureDetector, Gesture } from 'react-native-gesture-handler';
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';

function TappableCard() {
  const scale = useSharedValue(1);

  const tap = Gesture.Tap()
    .onBegin(() => { scale.value = withSpring(0.95); })
    .onFinalize(() => { scale.value = withSpring(1); })
    .onEnd(() => { runOnJS(handleTap)(); });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  return (
    <GestureDetector gesture={tap}>
      <Animated.View style={[styles.card, animatedStyle]} />
    </GestureDetector>
  );
}
```

## Pan Gesture (Draggable)

```tsx
function DraggableBox() {
  const offsetX = useSharedValue(0);
  const offsetY = useSharedValue(0);
  const startX = useSharedValue(0);
  const startY = useSharedValue(0);

  const pan = Gesture.Pan()
    .onStart(() => {
      startX.value = offsetX.value;
      startY.value = offsetY.value;
    })
    .onUpdate((event) => {
      offsetX.value = startX.value + event.translationX;
      offsetY.value = startY.value + event.translationY;
    })
    .onEnd(() => {
      offsetX.value = withSpring(0);
      offsetY.value = withSpring(0);
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      { translateX: offsetX.value },
      { translateY: offsetY.value },
    ],
  }));

  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={[styles.box, animatedStyle]} />
    </GestureDetector>
  );
}
```

## Pinch (Zoom)

```tsx
function ZoomableImage({ uri }: { uri: string }) {
  const scale = useSharedValue(1);
  const savedScale = useSharedValue(1);

  const pinch = Gesture.Pinch()
    .onUpdate((event) => {
      scale.value = savedScale.value * event.scale;
    })
    .onEnd(() => {
      savedScale.value = scale.value;
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  return (
    <GestureDetector gesture={pinch}>
      <Animated.Image source={{ uri }} style={[styles.image, animatedStyle]} />
    </GestureDetector>
  );
}
```

## Gesture Composition

```tsx
// Simultaneous — both gestures run at the same time
const composed = Gesture.Simultaneous(pan, pinch);

// Exclusive — first recognized gesture wins
const exclusive = Gesture.Exclusive(doubleTap, singleTap);

// Race — whichever gesture activates first cancels the others
const race = Gesture.Race(longPress, tap);
```

## Double Tap vs Single Tap

```tsx
const doubleTap = Gesture.Tap()
  .numberOfTaps(2)
  .onEnd(() => { runOnJS(handleDoubleTap)(); });

const singleTap = Gesture.Tap()
  .numberOfTaps(1)
  .requireExternalGestureToFail(doubleTap) // wait to confirm it's not a double tap
  .onEnd(() => { runOnJS(handleSingleTap)(); });

const gesture = Gesture.Exclusive(doubleTap, singleTap);
```

## LongPress

```tsx
const longPress = Gesture.LongPress()
  .minDuration(500)     // ms before activation
  .maxDistance(10)      // px movement before cancel
  .onStart(() => {
    runOnJS(triggerHaptic)();
    scale.value = withSpring(0.9);
  })
  .onEnd((_event, success) => {
    scale.value = withSpring(1);
    if (success) runOnJS(showContextMenu)();
  });
```

## Swipeable Row

```tsx
import { Swipeable } from 'react-native-gesture-handler';

function SwipeableRow({ children, onDelete }: SwipeableRowProps) {
  const renderRightActions = () => (
    <Pressable onPress={onDelete} style={styles.deleteAction}>
      <Text style={styles.deleteText}>Delete</Text>
    </Pressable>
  );

  return (
    <Swipeable
      renderRightActions={renderRightActions}
      rightThreshold={40}
      friction={2}
    >
      {children}
    </Swipeable>
  );
}
```

## hitSlop

Expand the touch target area beyond the visible bounds:

```tsx
const tap = Gesture.Tap()
  .hitSlop({ top: 10, bottom: 10, left: 10, right: 10 })
  .onEnd(() => { runOnJS(handleTap)(); });
```

## Platform Notes

- On Android, ensure `GestureHandlerRootView` wraps the entire app — partial wrapping causes silent failures
- Use `activeOffsetX`/`activeOffsetY` on `Pan` to delay activation and avoid conflicts with `ScrollView`:
  ```ts
  Gesture.Pan().activeOffsetX([-10, 10])
  ```
- `Gesture.Native()` allows native scroll views to participate in gesture competitions
