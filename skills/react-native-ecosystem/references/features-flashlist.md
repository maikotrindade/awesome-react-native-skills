# Features — FlashList v2

FlashList is a drop-in replacement for `FlatList` that **recycles** item views instead of creating and destroying them as you scroll. That gives steadier frame rates and less memory churn on long or complex lists. Version 2 was rewritten for the New Architecture and no longer needs size estimates.

> Targets `@shopify/flash-list` 2.x (New Architecture only). Apps still on the old architecture must stay on 1.x. For simple lists under ~50 items, built-in `FlatList` (see `react-native-core/features-lists.md`) is fine.

## Installation

```bash
npx expo install @shopify/flash-list   # Expo
npm install @shopify/flash-list         # bare RN
```

## Basic Usage

```tsx
import { FlashList } from '@shopify/flash-list';

export function Feed({ posts }: { posts: Post[] }) {
  return (
    <FlashList
      data={posts}
      keyExtractor={(post) => post.id}
      renderItem={({ item }) => <PostCard post={item} />}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
    />
  );
}
```

No `estimatedItemSize` and no fixed parent height are needed. FlashList measures items itself and sizes to its content or to the flex space its parent gives it.

## What Changed from v1

| v1 | v2 |
|----|----|
| `estimatedItemSize` required | Removed; sizes are measured automatically |
| `estimatedListSize`, `estimatedFirstItemOffset` | Removed |
| `MasonryFlashList` component | `<FlashList masonry />` |
| `overrideItemLayout` set `size` and `span` | Only `span` is used (for grids and masonry) |
| `inverted` for chat lists | Prefer `maintainVisibleContentPosition={{ startRenderingFromBottom: true }}` |
| `CellContainer` / `ref` type `FlashList<T>` | `ref` type is `FlashListRef<T>` |
| Old + New Architecture | New Architecture only |

When migrating, delete the size-estimate props first, then fix TypeScript errors on `ref`s.

## Recycling Rules

Because a view is **reused** for a different item, any state inside an item component would leak to the next item that reuses it.

```tsx
// ❌ Expanded state stays "true" when this view is recycled for another post
function PostCard({ post }: { post: Post }) {
  const [expanded, setExpanded] = useState(false);
  // ...
}

// ✅ Resets automatically when the item changes
import { useRecyclingState } from '@shopify/flash-list';

function PostCard({ post }: { post: Post }) {
  const [expanded, setExpanded] = useRecyclingState(false, [post.id]);
  // ...
}
```

- `useRecyclingState(initial, deps, onReset?)` acts like `useState` but resets whenever `deps` change.
- `useLayoutState(initial)` acts like `useState`, and also tells FlashList the item's size may change, so the list re-lays out without flicker (expand/collapse rows).
- When an item renders its own `.map()`, keep the inner keys stable with `useMappingHelper()` instead of using indices.
- Do **not** put `key` props on the item's root component. That defeats recycling.

## Mixed Item Types

Lists with very different row layouts (headers, ads, posts) recycle best when each type has its own pool:

```tsx
<FlashList
  data={rows}
  getItemType={(row) => row.type}   // 'header' | 'post' | 'ad'
  renderItem={({ item }) => {
    switch (item.type) {
      case 'header': return <SectionHeader title={item.title} />;
      case 'ad': return <AdSlot id={item.id} />;
      default: return <PostCard post={item} />;
    }
  }}
/>
```

## Grids and Masonry

```tsx
// fixed grid
<FlashList data={photos} numColumns={3} renderItem={renderPhoto} />

// Pinterest-style masonry (items of different heights)
<FlashList
  data={pins}
  masonry
  numColumns={2}
  optimizeItemArrangement   // places each item in the shortest column
  renderItem={renderPin}
/>

// let some items span several columns
<FlashList
  data={items}
  numColumns={4}
  overrideItemLayout={(layout, item) => { layout.span = item.featured ? 2 : 1; }}
  renderItem={renderTile}
/>
```

## Chat and Bidirectional Lists

`maintainVisibleContentPosition` is **on by default** in v2. When items are prepended, the visible content stays where it is instead of jumping:

```tsx
<FlashList
  data={messages}
  renderItem={renderMessage}
  onStartReached={loadOlderMessages}       // fires near the top
  maintainVisibleContentPosition={{
    startRenderingFromBottom: true,        // open at the newest message
    autoscrollToBottomThreshold: 0.2,      // stick to bottom when new messages arrive
    animateAutoScrollToBottom: true,
  }}
/>
```

Set `maintainVisibleContentPosition={{ disabled: true }}` for feeds where a jump is acceptable.

## Ref Methods

```tsx
import { FlashList, FlashListRef } from '@shopify/flash-list';

const listRef = useRef<FlashListRef<Post>>(null);

listRef.current?.scrollToIndex({ index: 0, animated: true });
listRef.current?.scrollToEnd({ animated: true });
listRef.current?.getFirstVisibleIndex();
```

## Animations

- Use `AnimatedFlashList` for Reanimated scroll handlers (`onScroll={useAnimatedScrollHandler(...)}`).
- Before removing or reordering items with a layout animation, call `listRef.current?.prepareForLayoutAnimationRender()` so FlashList doesn't recycle the animating views.

## Performance Checklist

- Keep `renderItem` components light and wrap them in `React.memo`; pass primitive props when possible.
- Use `getItemType` whenever rows differ structurally.
- Avoid inline `renderItem` closures that capture changing values. Pass changing values through `extraData` instead.
- Use `expo-image` (or another caching image component) with recycling-friendly props for image-heavy rows.
- Tune `drawDistance` (px rendered beyond the viewport) only after measuring. Larger values mean fewer blanks but more memory.
- Measure with the built-in `useBenchmark` hook, or compare against FlatList using `useFlatListBenchmark`, before and after changes.
