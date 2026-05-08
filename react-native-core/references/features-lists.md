---
name: react-native-lists
description: FlatList and SectionList for rendering performant lists. Use when displaying scrollable data, implementing infinite scroll, or creating sectioned lists.
---

# Lists

React Native provides `FlatList` and `SectionList` for efficiently rendering large, scrollable lists. They only render visible items, improving performance.

## FlatList

Renders flat lists with optional headers, footers, separators, and pull-to-refresh.

### Basic Usage

```tsx
import { FlatList, View, Text, StyleSheet } from 'react-native';

const DATA = [
  { id: '1', title: 'First Item' },
  { id: '2', title: 'Second Item' },
  { id: '3', title: 'Third Item' },
];

const App = () => (
  <FlatList
    data={DATA}
    renderItem={({ item }) => (
      <View style={styles.item}>
        <Text style={styles.title}>{item.title}</Text>
      </View>
    )}
    keyExtractor={item => item.id}
  />
);
```

### Required Props

```tsx
<FlatList
  data={items}                    // Array of items to render
  renderItem={({ item, index, separators }) => (
    <ItemComponent item={item} />
  )}
  keyExtractor={(item, index) => item.id} // Unique key for each item
/>
```

### Common Props

```tsx
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={keyExtractor}
  
  // Layout
  horizontal={false}              // Horizontal scrolling
  numColumns={1}                  // Multiple columns
  columnWrapperStyle={styles.row} // Style for column wrapper
  
  // Headers and Footers
  ListHeaderComponent={<Header />}
  ListFooterComponent={<Footer />}
  ListEmptyComponent={<EmptyState />}
  
  // Separators
  ItemSeparatorComponent={() => <View style={styles.separator} />}
  
  // Pull to Refresh
  refreshing={isRefreshing}
  onRefresh={() => refreshData()}
  refreshControl={<RefreshControl />}
  
  // Infinite Scroll
  onEndReached={() => loadMore()}
  onEndReachedThreshold={0.5}    // Trigger distance (0-1)
  
  // Performance
  initialNumToRender={10}        // Initial items to render
  maxToRenderPerBatch={10}       // Items per batch
  windowSize={21}                // Viewport multiplier
  removeClippedSubviews={true}   // Remove off-screen views
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}                            // Optimize for fixed item heights
  
  // Scroll Events
  onScroll={(event) => {
    const offsetY = event.nativeEvent.contentOffset.y;
  }}
  scrollEventThrottle={16}       // Throttle scroll events
  
  // Styling
  style={styles.list}
  contentContainerStyle={styles.content}
/>
```

### renderItem Function

Receives an object with item data and metadata:

```tsx
renderItem={({ item, index, separators }) => {
  // item: The data item from data array
  // index: Index of item in data array
  // separators: { highlight, unhighlight, updateProps }
  
  return <ItemComponent item={item} />;
}}
```

### keyExtractor

Extract unique key for each item. Required for proper list updates.

```tsx
// Simple ID
keyExtractor={(item) => item.id}

// Composite key
keyExtractor={(item, index) => `${item.id}-${index}`}

// Index (not recommended - breaks reordering)
keyExtractor={(item, index) => index.toString()}
```

### Horizontal Lists

```tsx
<FlatList
  data={items}
  renderItem={renderItem}
  horizontal={true}
  showsHorizontalScrollIndicator={true}
  ItemSeparatorComponent={() => <View style={{ width: 10 }} />}
/>
```

### Multiple Columns

```tsx
<FlatList
  data={items}
  renderItem={renderItem}
  numColumns={2}
  columnWrapperStyle={styles.row}
  keyExtractor={(item, index) => `${item.id}-${index}`}
/>

const styles = StyleSheet.create({
  row: {
    flex: 1,
    justifyContent: 'space-around',
  },
});
```

### Pull to Refresh

```tsx
const [refreshing, setRefreshing] = useState(false);

const onRefresh = async () => {
  setRefreshing(true);
  await fetchData();
  setRefreshing(false);
};

<FlatList
  refreshing={refreshing}
  onRefresh={onRefresh}
  // ... other props
/>
```

### Infinite Scroll

```tsx
const [data, setData] = useState(initialData);
const [loading, setLoading] = useState(false);

const loadMore = async () => {
  if (loading) return;
  
  setLoading(true);
  const moreData = await fetchMore();
  setData([...data, ...moreData]);
  setLoading(false);
};

<FlatList
  data={data}
  onEndReached={loadMore}
  onEndReachedThreshold={0.5}
  ListFooterComponent={loading ? <LoadingSpinner /> : null}
/>
```

### Performance Optimization

```tsx
// For fixed-height items
<FlatList
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
  // ... other props
/>

// Reduce initial render
<FlatList
  initialNumToRender={10}
  maxToRenderPerBatch={10}
  windowSize={10} // Smaller = less memory
  removeClippedSubviews={true}
/>
```

## SectionList

Renders sectioned lists with section headers and optional section footers.

### Basic Usage

```tsx
import { SectionList, View, Text, StyleSheet } from 'react-native';

const DATA = [
  {
    title: 'Section 1',
    data: ['Item 1', 'Item 2'],
  },
  {
    title: 'Section 2',
    data: ['Item 3', 'Item 4'],
  },
];

const App = () => (
  <SectionList
    sections={DATA}
    renderItem={({ item }) => (
      <View style={styles.item}>
        <Text>{item}</Text>
      </View>
    )}
    renderSectionHeader={({ section: { title } }) => (
      <Text style={styles.header}>{title}</Text>
    )}
    keyExtractor={(item, index) => item + index}
  />
);
```

### Required Props

```tsx
<SectionList
  sections={sections}            // Array of section objects
  renderItem={({ item, index, section }) => (
    <ItemComponent item={item} />
  )}
  renderSectionHeader={({ section: { title } }) => (
    <HeaderComponent title={title} />
  )}
  keyExtractor={(item, index) => item.id}
/>
```

### Section Data Structure

```tsx
const sections = [
  {
    title: 'Section Title',
    data: [item1, item2, item3], // Items in this section
    renderItem: customRenderItem, // Optional: override default
    key: 'section-key',          // Optional: unique key
  },
];
```

### Common Props

```tsx
<SectionList
  sections={sections}
  renderItem={renderItem}
  renderSectionHeader={renderSectionHeader}
  renderSectionFooter={({ section }) => <Footer />} // Optional
  keyExtractor={keyExtractor}
  
  // Sticky Headers
  stickySectionHeadersEnabled={true}
  
  // Same props as FlatList
  ListHeaderComponent={<Header />}
  ListFooterComponent={<Footer />}
  onEndReached={loadMore}
  refreshing={refreshing}
  onRefresh={onRefresh}
/>
```

### renderSectionHeader

```tsx
renderSectionHeader={({ section: { title, data } }) => (
  <View style={styles.header}>
    <Text>{title}</Text>
    <Text>{data.length} items</Text>
  </View>
)}
```

## Best Practices

1. **Always provide keyExtractor** - Required for proper list updates
2. **Use getItemLayout** - For fixed-height items to improve performance
3. **Optimize renderItem** - Use React.memo for item components
4. **Set initialNumToRender** - Balance initial load time vs memory
5. **Use removeClippedSubviews** - Remove off-screen views (Android)
6. **Throttle scroll events** - Use scrollEventThrottle for performance
7. **Handle empty states** - Use ListEmptyComponent
8. **Implement pagination** - Use onEndReached for infinite scroll
9. **Debounce onEndReached** - Prevent multiple calls
10. **Test with large datasets** - Ensure performance with real data

## Common Patterns

### Searchable List

```tsx
const [searchQuery, setSearchQuery] = useState('');
const filteredData = data.filter(item =>
  item.title.toLowerCase().includes(searchQuery.toLowerCase())
);

<FlatList
  data={filteredData}
  renderItem={renderItem}
  keyExtractor={keyExtractor}
/>
```

### Grouped List

```tsx
const grouped = data.reduce((acc, item) => {
  const key = item.category;
  if (!acc[key]) acc[key] = [];
  acc[key].push(item);
  return acc;
}, {});

const sections = Object.entries(grouped).map(([title, data]) => ({
  title,
  data,
}));

<SectionList sections={sections} ... />
```

### Selectable List

```tsx
const [selectedIds, setSelectedIds] = useState(new Set());

const toggleSelection = (id) => {
  setSelectedIds(prev => {
    const next = new Set(prev);
    if (next.has(id)) next.delete(id);
    else next.add(id);
    return next;
  });
};

renderItem={({ item }) => (
  <Pressable onPress={() => toggleSelection(item.id)}>
    <ItemComponent
      item={item}
      selected={selectedIds.has(item.id)}
    />
  </Pressable>
)}
```

<!--
Source references:
- https://reactnative.dev/docs/flatlist
- https://reactnative.dev/docs/sectionlist
- https://reactnative.dev/docs/virtualizedlist
-->
