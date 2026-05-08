---
name: react-native-layout-flexbox
description: Layout with Flexbox in React Native. Use when positioning components, creating responsive layouts, or aligning content.
---

# Layout with Flexbox

React Native uses Flexbox for layout. The defaults differ from web CSS: `flexDirection` defaults to `column` instead of `row`, and `flexShrink` defaults to `0` instead of `1`.

## Flex Direction

Controls the direction children are laid out (main axis).

```tsx
import { View, StyleSheet } from 'react-native';

const App = () => (
  <View style={{ flexDirection: 'row' }}>
    <View style={styles.box} />
    <View style={styles.box} />
  </View>
);
```

**Values:**
- `column` (default) - Top to bottom
- `row` - Left to right
- `column-reverse` - Bottom to top
- `row-reverse` - Right to left

## Flex

Defines how items fill available space along the main axis.

```tsx
<View style={{ flex: 1 }}>
  <View style={{ flex: 1, backgroundColor: 'red' }} />      {/* 1/6 of space */}
  <View style={{ flex: 2, backgroundColor: 'blue' }} />    {/* 2/6 of space */}
  <View style={{ flex: 3, backgroundColor: 'green' }} />  {/* 3/6 of space */}
</View>
```

Space is divided proportionally: `1 + 2 + 3 = 6`, so each gets 1/6, 2/6, and 3/6 respectively.

## Justify Content

Aligns children along the main axis.

```tsx
<View style={{ justifyContent: 'center' }}>
  {/* Children centered horizontally (if row) or vertically (if column) */}
</View>
```

**Values:**
- `flex-start` (default) - Start of container
- `flex-end` - End of container
- `center` - Center of container
- `space-between` - Space between items
- `space-around` - Space around items
- `space-evenly` - Equal space everywhere

## Align Items

Aligns children along the cross axis (perpendicular to main axis).

```tsx
<View style={{ flexDirection: 'row', alignItems: 'center' }}>
  {/* Children centered vertically */}
</View>
```

**Values:**
- `stretch` (default) - Stretch to match container height
- `flex-start` - Start of cross axis
- `flex-end` - End of cross axis
- `center` - Center of cross axis
- `baseline` - Align to baseline

**Note**: `stretch` requires children to not have fixed dimensions on the cross axis.

## Align Self

Overrides parent's `alignItems` for a single child.

```tsx
<View style={{ alignItems: 'stretch' }}>
  <View style={{ alignSelf: 'flex-end' }}>
    {/* This child overrides parent alignment */}
  </View>
</View>
```

## Align Content

Distributes wrapped lines along the cross axis. Only works with `flexWrap: 'wrap'`.

```tsx
<View style={{ flexWrap: 'wrap', alignContent: 'space-between' }}>
  {/* Wrapped lines distributed */}
</View>
```

**Values:**
- `flex-start` (default) - Start of cross axis
- `flex-end` - End of cross axis
- `stretch` - Stretch lines
- `center` - Center lines
- `space-between` - Space between lines
- `space-around` - Space around lines
- `space-evenly` - Equal space everywhere

## Flex Wrap

Controls whether children wrap to multiple lines.

```tsx
<View style={{ flexWrap: 'wrap' }}>
  {/* Children wrap when they overflow */}
</View>
```

**Values:**
- `nowrap` (default) - Single line, may shrink
- `wrap` - Wrap to multiple lines

## Flex Basis, Grow, and Shrink

Fine-grained control over flex behavior.

```tsx
<View style={{
  flexBasis: 100,    // Default size before grow/shrink
  flexGrow: 1,       // How much to grow (default: 0)
  flexShrink: 1      // How much to shrink (default: 0)
}} />
```

- **flexBasis**: Default size along main axis (like width/height)
- **flexGrow**: Distribute remaining space proportionally
- **flexShrink**: Shrink proportionally when space is limited

## Gap

Sets spacing between rows and columns.

```tsx
<View style={{
  flexWrap: 'wrap',
  gap: 10,           // Both row and column gap
  rowGap: 10,         // Row gap only
  columnGap: 10       // Column gap only
}}>
  {/* Consistent spacing between items */}
</View>
```

## Width and Height

```tsx
<View style={{
  width: 100,              // Pixels
  width: '50%',            // Percentage of parent
  width: 'auto',           // Calculate from content (default)
  height: 200,
  minWidth: 50,
  maxWidth: 300
}} />
```

**Values:**
- `auto` (default) - Calculated from content
- Number - Absolute pixels
- String percentage - `'50%'` relative to parent

## Position

Controls positioning relative to containing block.

```tsx
<View style={{
  position: 'relative',  // Default - normal flow
  position: 'absolute',  // Absolute positioning
  position: 'static',    // New Architecture only
  top: 10,
  left: 20,
  right: 30,
  bottom: 40
}} />
```

**Values:**
- `relative` (default) - Normal flow with offset
- `absolute` - Positioned relative to containing block
- `static` - Normal flow, ignore top/left/right/bottom

### Containing Block

The containing block determines positioning context for absolute elements:

- **relative/static**: Containing block is parent
- **absolute**: Containing block is nearest ancestor with:
  - `position` other than `static`, OR
  - `transform` property

## Layout Direction

Controls text and layout direction (LTR/RTL).

```tsx
<View style={{ direction: 'rtl' }}>
  {/* Right-to-left layout */}
</View>
```

**Values:**
- `ltr` (default) - Left to right
- `rtl` - Right to left

Affects `start`/`end` values in margin/padding.

## Common Layout Patterns

### Centered Content

```tsx
<View style={{
  flex: 1,
  justifyContent: 'center',
  alignItems: 'center'
}}>
  <Text>Centered</Text>
</View>
```

### Row with Space Between

```tsx
<View style={{
  flexDirection: 'row',
  justifyContent: 'space-between',
  alignItems: 'center'
}}>
  <Text>Left</Text>
  <Text>Right</Text>
</View>
```

### Equal Width Columns

```tsx
<View style={{ flexDirection: 'row' }}>
  <View style={{ flex: 1 }} />
  <View style={{ flex: 1 }} />
  <View style={{ flex: 1 }} />
</View>
```

### Sticky Footer

```tsx
<View style={{ flex: 1 }}>
  <View style={{ flex: 1 }}>
    {/* Scrollable content */}
  </View>
  <View>
    {/* Footer always at bottom */}
  </View>
</View>
```

## Key Differences from Web CSS

1. **flexDirection**: Defaults to `column` (not `row`)
2. **alignContent**: Defaults to `flex-start` (not `stretch`)
3. **flexShrink**: Defaults to `0` (not `1`)
4. **flex**: Only accepts single number (not `1 1 auto`)
5. **No CSS Grid**: Use Flexbox for all layouts
6. **No float**: Use Flexbox positioning

## Best Practices

1. **Use `flex: 1`** to fill available space
2. **Set explicit dimensions** for images and fixed-size elements
3. **Use `flexWrap: 'wrap'`** with `gap` for grid layouts
4. **Avoid absolute positioning** when Flexbox can solve it
5. **Test on different screen sizes** - use percentages and flex for responsiveness
6. **Use `alignSelf`** to override parent alignment for specific children

<!--
Source references:
- https://reactnative.dev/docs/flexbox
- https://reactnative.dev/docs/layout-props
- https://www.yogalayout.dev/
-->
