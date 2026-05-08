---
name: react-native-performance
description: Performance optimization for React Native apps — Hermes, New Architecture, FlatList tuning, memoization, images, and profiling tools.
---

# Performance Optimization

React Native targets 60fps (16.67ms/frame). Work that exceeds this budget on the JS thread or UI thread causes dropped frames and jank.

- **JS thread**: React reconciliation, state updates, event handlers, API calls
- **UI thread**: Native view rendering, animations (with `useNativeDriver` or Reanimated)

---

## Hermes Engine

Hermes is the default JS engine since RN 0.70+. It AOT-compiles JS to bytecode at build time, reducing startup time by 30–50% compared to JSC.

Verify Hermes is active:
```tsx
import { HermesInternal } from 'global';
console.log('Running on Hermes:', !!HermesInternal); // should be true
```

No configuration needed — Hermes is on by default.

---

## New Architecture (RN 0.76+)

New Architecture is **enabled by default** since RN 0.76. It brings:

- **Fabric** renderer — layout runs on UI thread, synchronous `measure()`, React 18 concurrent features
- **TurboModules** — lazy-loaded native modules, synchronous JS↔native calls via JSI
- **No JSON bridge** — native ↔ JS communication is direct C++ (JSI), eliminating serialization overhead

**For performance**: Reanimated v3 and Gesture Handler v2 already use JSI/Fabric — upgrade both if you haven't. Check third-party library compatibility at [reactnative.directory](https://reactnative.directory).

---

## FlatList Optimization

```tsx
<FlatList
  data={items}
  keyExtractor={(item) => item.id}       // stable, cheap key — never use index

  // Render tuning
  initialNumToRender={10}                 // items on first paint
  maxToRenderPerBatch={5}                 // items added per render cycle
  windowSize={5}                          // render 5× viewport height
  updateCellsBatchingPeriod={50}          // ms between batches

  removeClippedSubviews={true}            // unmount off-screen items (Android)

  // Skip layout calculation for fixed-height rows
  getItemLayout={(_, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}

  renderItem={renderItem}                 // stable ref — defined outside render or useCallback
/>
```

Always memoize `renderItem`:
```tsx
const renderItem = useCallback(
  ({ item }: { item: Post }) => <PostCard post={item} />,
  []
);
```

---

## Memoization

Apply memoization where a component re-renders frequently with unchanged props:

```tsx
// React.memo — skip re-render if props are shallowly equal
const PostCard = React.memo(function PostCard({ post }: { post: Post }) {
  return <View>{/* ... */}</View>;
});

// useCallback — stable function reference for child components
const handlePress = useCallback(() => {
  navigation.navigate('Detail', { id: item.id });
}, [item.id]);

// useMemo — expensive derived values
const sorted = useMemo(
  () => [...items].sort((a, b) => b.createdAt - a.createdAt),
  [items]
);
```

Memoization is not free — only apply it when you can measure the benefit. A good heuristic: memoize components with many props or that are rendered inside a large list.

---

## Avoid Common Re-render Causes

```tsx
// Bad — new object on every render
<Component style={{ marginTop: 10 }} />

// Good — stable StyleSheet reference
const styles = StyleSheet.create({ wrapper: { marginTop: 10 } });
<Component style={styles.wrapper} />

// Bad — anonymous callback
<Button onPress={() => handlePress(item.id)} />

// Good — stable callback
const onPress = useCallback(() => handlePress(item.id), [item.id]);
<Button onPress={onPress} />
```

---

## Images

Prefer **`expo-image`** over the built-in `<Image>`:
- Memory + disk cache
- Progressive loading, blurhash placeholders
- Faster decoding, smooth transitions

```tsx
import { Image } from 'expo-image';

<Image
  source={{ uri: photo.url }}
  style={{ width: 100, height: 100 }}
  placeholder={photo.blurhash}
  contentFit="cover"
  transition={200}
  cachePolicy="memory-disk"
/>
```

Always set explicit `width` and `height` — prevents layout recalculation.

---

## InteractionManager

Defer heavy work until after navigation/animation transitions complete:

```tsx
import { InteractionManager } from 'react-native';

useFocusEffect(
  useCallback(() => {
    const task = InteractionManager.runAfterInteractions(() => {
      loadHeavyData();
    });
    return () => task.cancel();
  }, [])
);
```

---

## Development vs Release Performance

Always profile in **release mode** — development mode adds significant overhead (warnings, error checking):

```bash
# Android release build
./gradlew assembleRelease

# iOS — build Release scheme in Xcode
```

Remove console statements in production:
```json
// babel.config.js env.production
{
  "plugins": ["transform-remove-console"]
}
```

---

## Profiling Tools

| Tool | Purpose |
|------|---------|
| **Perf Monitor** | In-app FPS, JS FPS, RAM — shake device → Perf Monitor |
| **React DevTools Profiler** | Component render times, what triggered re-renders |
| **React Native DevTools** | Built-in DevTools (RN 0.76+) — no Flipper needed |
| **Xcode Instruments** | CPU, memory, GPU on iOS |
| **Android Studio Profiler** | CPU, memory, network on Android |

Enable Perf Monitor in app:
```tsx
import { DevSettings } from 'react-native';
// Accessible from Dev Menu in debug builds
```

---

## Bundle Size

```bash
# Expo: analyze bundle
npx expo export --dump-sourcemap
npx source-map-explorer dist/bundles/index.ios.js --html report.html
```

- Import only what you need: `import debounce from 'lodash/debounce'` not `import _ from 'lodash'`
- Use Metro's tree shaking for ES modules
- Use dynamic `require()` for rarely-used heavy modules
