# Best Practices — Performance

## JS Thread vs UI Thread

React Native has two main threads:
- **JS thread** — runs your JavaScript, React reconciliation, state updates
- **UI thread** (main thread) — renders native views, handles touches

Jank occurs when the JS thread is blocked and can't deliver frames at 60fps. Heavy computations, large re-renders, and synchronous storage reads all block the JS thread.

---

## FlatList Optimization

```tsx
<FlatList
  data={items}
  keyExtractor={(item) => item.id}            // stable, cheap key

  // Tuning props
  initialNumToRender={10}                      // items rendered on first paint
  maxToRenderPerBatch={5}                      // items added per render cycle
  windowSize={5}                               // render window = 5 × viewport height
  updateCellsBatchingPeriod={50}               // ms between batched renders
  removeClippedSubviews={true}                 // unmount off-screen items (Android)

  // If all items have known fixed height — enables O(1) scroll position
  getItemLayout={(_, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}

  renderItem={renderItem}                      // must be stable reference
/>
```

Always memoize `renderItem` to prevent full-list re-renders:

```tsx
const renderItem = useCallback(
  ({ item }: { item: Post }) => <PostCard post={item} />,
  [] // no dependencies if PostCard is memoized
);
```

---

## Memoization

```tsx
// React.memo — skip re-render if props are shallowly equal
const PostCard = React.memo(function PostCard({ post }: { post: Post }) {
  return <View>{/* ... */}</View>;
});

// Custom comparison (when shallow equality isn't enough)
const PostCard = React.memo(PostCardComponent, (prev, next) => {
  return prev.post.id === next.post.id && prev.post.likeCount === next.post.likeCount;
});

// useCallback — stable function reference
const handlePress = useCallback(() => {
  navigation.navigate('Detail', { id: item.id });
}, [item.id, navigation]);

// useMemo — expensive derived value
const sortedItems = useMemo(
  () => [...items].sort((a, b) => b.createdAt - a.createdAt),
  [items]
);
```

Memoization is not free — only apply it when:
- A component re-renders frequently with the same props
- A computation is measurably expensive
- A function is passed as a prop to a memoized child

---

## Hermes Engine

Hermes is the default JS engine since RN 0.70+. It pre-compiles JS to bytecode at build time, reducing startup time significantly. No action needed — it's on by default.

Verify Hermes is active:
```ts
import { HermesInternal } from 'global';
console.log('Hermes:', !!HermesInternal); // should be true
```

---

## Images

Use `expo-image` or `react-native-fast-image` instead of the built-in `<Image>`:
- Disk + memory caching
- Better placeholder support
- Faster decoding

```tsx
import { Image } from 'expo-image';

<Image
  source={{ uri: user.avatarUrl }}
  style={styles.avatar}
  placeholder={require('./assets/avatar-placeholder.png')}
  contentFit="cover"
  transition={200}
  cachePolicy="memory-disk"
/>
```

Always set explicit `width` and `height` — avoids layout recalculation.

---

## InteractionManager

Defer expensive work until after animations/transitions complete:

```ts
import { InteractionManager } from 'react-native';

useFocusEffect(
  useCallback(() => {
    const task = InteractionManager.runAfterInteractions(() => {
      loadHeavyData(); // runs after screen transition animation ends
    });
    return () => task.cancel();
  }, [])
);
```

---

## Avoid Common Re-render Causes

```tsx
// Bad — new object created on every render
<Component style={{ marginTop: 10 }} />

// Good — stable reference
const styles = StyleSheet.create({ wrapper: { marginTop: 10 } });
<Component style={styles.wrapper} />

// Bad — anonymous callback creates new reference every render
<Button onPress={() => doSomething(id)} />

// Good — stable reference
const handlePress = useCallback(() => doSomething(id), [id]);
<Button onPress={handlePress} />

// Bad — Context causes all consumers to re-render on any value change
const AppContext = createContext({ theme, user, cart }); // huge object

// Good — split contexts by update frequency
const ThemeContext = createContext(theme);
const UserContext = createContext(user);
```

---

## Bundle Size

```bash
# Analyze bundle with Metro
npx react-native bundle --platform ios --dev false --entry-file index.js \
  --bundle-output /tmp/bundle.js --assets-dest /tmp/assets

# With Expo
npx expo export --dump-sourcemap
npx source-map-explorer dist/bundles/index.ios.js --html report.html
```

- Avoid importing entire libraries when you only need a part: `import { debounce } from 'lodash/debounce'` not `import _ from 'lodash'`
- Use `babel-plugin-transform-imports` or `babel-plugin-lodash` for tree-shaking libraries that don't support it natively

---

## Profiling Tools

| Tool | What it shows |
|------|--------------|
| **React DevTools Profiler** | Component render durations, which components re-rendered |
| **Perf Monitor** (in-app) | JS FPS, UI FPS, RAM usage — shake device → Perf Monitor |
| **Flipper** | Network, layout, crash logs, Hermes profiler |
| **Systrace** (Android) | Native thread activity, frame timing |
| **Instruments** (iOS Xcode) | CPU, memory, GPU usage |

Enable Perf Monitor in development:
```ts
import { DevSettings } from 'react-native';
DevSettings.addMenuItem('Toggle Perf Monitor', () => { /* opens overlay */ });
```

---

## New Architecture Performance Benefits

With New Architecture enabled (default in RN 0.76+):
- **Synchronous layout reads** — no async bridge round-trips for measurements
- **Concurrent rendering** — React 18 concurrent features work natively
- **Fabric** renders on UI thread directly, eliminating shadow tree sync
- **TurboModules** — native modules loaded lazily, synchronous JS calls possible

See `advanced-new-architecture.md` for migration details.
