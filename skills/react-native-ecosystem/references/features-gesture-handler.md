# Features — React Native Gesture Handler 3

Gesture Handler recognizes touches with each platform's native gesture system, so gestures stay responsive even when the JS thread is busy. Version 3 replaces the builder chain (`Gesture.Pan().onUpdate(...)`) with **hooks** that take a config object.

> Targets `react-native-gesture-handler` 3.x (New Architecture). The v2 builder API (`Gesture.*`) is still exported for migration, and the old v2 components are exported with a `Legacy` prefix (`LegacyPressable`, `LegacyScrollView`, …). Write new code with the hook API.

## Installation

```bash
npx expo install react-native-gesture-handler   # Expo
npm install react-native-gesture-handler         # bare RN
```

Wrap the app root once, as the outermost view:

```tsx
import { GestureHandlerRootView } from 'react-native-gesture-handler';

export default function App() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      {/* navigation, providers, screens */}
    </GestureHandlerRootView>
  );
}
```

Expo Router apps put this in `app/_layout.tsx`. Modals that render outside the main tree (React Native `Modal`) need their own `GestureHandlerRootView`.

## Gesture Hooks

| Hook | Recognizes |
|------|------------|
| `useTapGesture` | Single or multi-tap (`numberOfTaps`, `maxDelay`, `maxDistance`) |
| `useLongPressGesture` | Press-and-hold (`minDuration`, `maxDistance`) |
| `usePanGesture` | Drag (`activeOffsetX/Y`, `failOffsetX/Y`, `minPointers`) |
| `usePinchGesture` | Two-finger zoom (`scale`, `focalX/Y`) |
| `useRotationGesture` | Two-finger rotate (`rotation`) |
| `useFlingGesture` | Quick swipe in a `direction` |
| `useHoverGesture` | Pointer hover (iPad, web, Android with mouse) |
| `useNativeGesture` | Lets a native scrollable take part in gesture relations |
| `useManualGesture` | You decide when it activates or fails |

Attach any gesture with `GestureDetector`:

```tsx
import { GestureDetector, useTapGesture } from 'react-native-gesture-handler';
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';
import { scheduleOnRN } from 'react-native-worklets';

function PressableCard({ onOpen }: { onOpen: () => void }) {
  const scale = useSharedValue(1);

  const tap = useTapGesture({
    onBegin: () => { scale.value = withSpring(0.96); },
    onFinalize: () => { scale.value = withSpring(1); },
    onDeactivate: (e) => {
      if (!e.canceled) scheduleOnRN(onOpen);
    },
  });

  const style = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }));

  return (
    <GestureDetector gesture={tap}>
      <Animated.View style={[styles.card, style]} />
    </GestureDetector>
  );
}
```

## Callback Lifecycle

| v3 callback | v2 equivalent | Fires when |
|-------------|---------------|------------|
| `onBegin` | `onBegin` | A pointer touches down and the gesture starts tracking |
| `onActivate` | `onStart` | The gesture is recognized |
| `onUpdate` | `onUpdate` / `onChange` | Every update while active |
| `onDeactivate` | `onEnd` | An active gesture ends; `e.canceled` is `true` if it did not complete |
| `onFinalize` | `onFinalize` | Always last, whether or not the gesture activated |
| `onTouchesDown/Move/Up/Cancel` | same | Raw touch events |

With Reanimated installed, callbacks run **on the UI thread as worklets**. Update shared values directly, and use `scheduleOnRN` to call React state setters or navigation. Set `runOnJS: true` in the config to run callbacks on the JS thread instead (e.g., for a gesture that only calls React code), or `disableReanimated: true` to opt out of Reanimated integration entirely.

## Pan (Draggable)

```tsx
function Draggable() {
  const x = useSharedValue(0);
  const y = useSharedValue(0);
  const startX = useSharedValue(0);
  const startY = useSharedValue(0);

  const pan = usePanGesture({
    onActivate: () => {
      startX.value = x.value;
      startY.value = y.value;
    },
    onUpdate: (e) => {
      x.value = startX.value + e.translationX;
      y.value = startY.value + e.translationY;
    },
    onDeactivate: (e) => {
      x.value = withDecay({ velocity: e.velocityX });
      y.value = withDecay({ velocity: e.velocityY });
    },
  });

  const style = useAnimatedStyle(() => ({
    transform: [{ translateX: x.value }, { translateY: y.value }],
  }));

  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={[styles.box, style]} />
    </GestureDetector>
  );
}
```

Inside a vertical `ScrollView`, give a horizontal pan `activeOffsetX: [-10, 10]` and `failOffsetY: [-5, 5]` so vertical scrolling still wins.

## Composing Gestures

Combine several gestures on one view with composition hooks:

```tsx
const pinch = usePinchGesture({ /* ... */ });
const rotate = useRotationGesture({ /* ... */ });
const both = useSimultaneousGestures(pinch, rotate);   // v2: Gesture.Simultaneous

const doubleTap = useTapGesture({ numberOfTaps: 2, onDeactivate: zoomIn });
const singleTap = useTapGesture({ onDeactivate: toggleControls });
const taps = useExclusiveGestures(doubleTap, singleTap); // v2: Gesture.Exclusive — first in the list has priority

const race = useCompetingGestures(longPress, pan);       // v2: Gesture.Race — first to activate wins

<GestureDetector gesture={taps}>…</GestureDetector>
```

Relate gestures that live on **different** views with config keys:

| Key | Meaning | v2 method |
|-----|---------|-----------|
| `simultaneousWith` | Can be active at the same time as these gestures | `.simultaneousWithExternalGesture()` |
| `requireToFail` | Waits for these gestures to fail before activating | `.requireExternalGestureToFail()` |
| `block` | Prevents these gestures from activating while this one can | `.blocksExternalGesture()` |

```tsx
const native = useNativeGesture({});
const pan = usePanGesture({ simultaneousWith: native, /* ... */ });
```

## Config Values Can Be Shared Values

Gesture config accepts Reanimated shared values, so you can enable or disable a gesture from the UI thread without re-rendering:

```tsx
const enabled = useSharedValue(true);
const pan = usePanGesture({ enabled, onUpdate: (e) => { /* ... */ } });
```

## Built-in Components

| Component | Use |
|-----------|-----|
| `Pressable` | Drop-in for React Native `Pressable`, handled natively so it composes with gestures |
| `RectButton`, `BorderlessButton` | Buttons with native ripple/highlight feedback |
| `ReanimatedSwipeable` | Swipe-to-reveal rows. Import from `react-native-gesture-handler/ReanimatedSwipeable` |
| `ReanimatedDrawerLayout` | Cross-platform drawer. Import from `react-native-gesture-handler/ReanimatedDrawerLayout` |

```tsx
import ReanimatedSwipeable from 'react-native-gesture-handler/ReanimatedSwipeable';

<ReanimatedSwipeable
  friction={2}
  rightThreshold={40}
  renderRightActions={(progress, translation) => <DeleteAction progress={progress} />}
>
  <Row />
</ReanimatedSwipeable>
```

The non-Reanimated `Swipeable` from v2 is gone. Use `ReanimatedSwipeable`.

## Migrating from v2

| v2 | v3 |
|----|----|
| `Gesture.Pan().minDistance(10).onUpdate(fn)` | `usePanGesture({ minDistance: 10, onUpdate: fn })` |
| `.onStart(fn)` | `onActivate: fn` |
| `.onEnd((e, success) => …)` | `onDeactivate: (e) => …` with `!e.canceled` |
| `.onChange(fn)` | `onUpdate: fn` (events include `changeX`/`changeY`) |
| `Gesture.Simultaneous(a, b)` | `useSimultaneousGestures(a, b)` |
| `Gesture.Exclusive(a, b)` | `useExclusiveGestures(a, b)` |
| `Gesture.Race(a, b)` | `useCompetingGestures(a, b)` |
| `.simultaneousWithExternalGesture(g)` | `simultaneousWith: g` |
| `.runOnJS(true)` | `runOnJS: true` |
| `runOnJS(fn)()` inside callbacks | `scheduleOnRN(fn)` from `react-native-worklets` |
| `Pressable`, `ScrollView`, `FlatList` from v2 | Renamed `LegacyPressable`, `LegacyScrollView`, …; prefer the new `Pressable` |

Migrate one screen at a time: the legacy builder API keeps working alongside hooks during the transition.

## Testing

Use the Jest setup shipped with the package and fire gestures by `testID`:

```js
// jest.config.js
setupFiles: ['./node_modules/react-native-gesture-handler/jestSetup.js'],
```

```tsx
import { fireGestureHandler, getByGestureTestId } from 'react-native-gesture-handler/jest-utils';

const pan = usePanGesture({ testID: 'card-pan', onDeactivate });
fireGestureHandler(getByGestureTestId('card-pan'), [{ translationX: 0 }, { translationX: 120 }]);
```

## Platform Notes

- `GestureHandlerRootView` must wrap the whole app. Gestures outside it silently do nothing, especially on Android.
- Gesture hooks follow the rules of hooks: create them in the component body, never conditionally. Disable a gesture with `enabled` instead of not creating it.
- `hitSlop` expands the touch area; use it for small icons instead of padding the view.
- On web, `userSelect`, `touchAction`, and `enableContextMenu` are props on `GestureDetector`.
