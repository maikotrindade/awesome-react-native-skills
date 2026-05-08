---
name: react-native-styling-stylesheet
description: Styling components with StyleSheet API. Use when creating styles, optimizing performance, or organizing component styles.
---

# Styling with StyleSheet

StyleSheet provides an abstraction similar to CSS stylesheets. It offers performance optimizations and better code organization than inline styles.

## Basic Usage

```tsx
import { StyleSheet, View, Text } from 'react-native';

const App = () => (
  <View style={styles.container}>
    <Text style={styles.title}>React Native</Text>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    backgroundColor: '#eaeaea',
  },
  title: {
    fontSize: 30,
    fontWeight: 'bold',
    color: '#20232a',
  },
});
```

## StyleSheet.create()

Creates an optimized style object. Styles are only created once and referenced by ID.

```tsx
const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  text: {
    fontSize: 16,
  },
});

// Use styles
<View style={styles.container} />
<Text style={styles.text} />
```

### Benefits

- **Performance**: Styles are created once, not on every render
- **Type safety**: IDEs provide autocomplete and type checking
- **Organization**: Separates styles from component logic
- **Validation**: Catches invalid style properties at creation time

## Style Composition

Combine multiple styles using arrays.

```tsx
const styles = StyleSheet.create({
  base: {
    padding: 10,
    backgroundColor: '#fff',
  },
  highlighted: {
    backgroundColor: '#f0f0f0',
  },
  text: {
    fontSize: 16,
  },
  bold: {
    fontWeight: 'bold',
  },
});

// Later styles override earlier ones
<View style={[styles.base, styles.highlighted]} />
<Text style={[styles.text, styles.bold, { color: 'red' }]} />
```

### StyleSheet.compose()

Combines two styles, with the second overriding the first.

```tsx
const page = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    backgroundColor: '#fff',
  },
});

const lists = StyleSheet.create({
  container: {
    backgroundColor: '#61dafb',
  },
});

// Compose styles
const container = StyleSheet.compose(page.container, lists.container);
```

**Note**: Returns the other style if one is falsy, maintaining reference equality for PureComponent checks.

## Conditional Styles

Use arrays with conditional logic for dynamic styles.

```tsx
const App = ({ isActive }) => (
  <View style={[
    styles.button,
    isActive && styles.active,
    { opacity: isActive ? 1 : 0.5 }
  ]} />
);

const styles = StyleSheet.create({
  button: {
    padding: 10,
    borderRadius: 5,
  },
  active: {
    backgroundColor: 'blue',
  },
});
```

## Style Properties

React Native supports a subset of CSS properties.

### Layout Properties

```tsx
{
  width: 100,
  height: 100,
  minWidth: 50,
  maxWidth: 300,
  minHeight: 50,
  maxHeight: 300,
  flex: 1,
  flexDirection: 'row',
  flexWrap: 'wrap',
  justifyContent: 'center',
  alignItems: 'center',
  alignSelf: 'flex-start',
  position: 'relative',
  top: 10,
  left: 20,
  right: 30,
  bottom: 40,
  margin: 10,
  marginVertical: 10,
  marginHorizontal: 10,
  marginTop: 10,
  marginBottom: 10,
  marginLeft: 10,
  marginRight: 10,
  padding: 10,
  paddingVertical: 10,
  paddingHorizontal: 10,
  paddingTop: 10,
  paddingBottom: 10,
  paddingLeft: 10,
  paddingRight: 10,
}
```

### Appearance Properties

```tsx
{
  backgroundColor: '#fff',
  opacity: 0.8,
  borderRadius: 5,
  borderWidth: 1,
  borderColor: '#000',
  borderTopWidth: 1,
  borderTopColor: '#000',
  borderBottomWidth: 1,
  borderBottomColor: '#000',
  borderLeftWidth: 1,
  borderLeftColor: '#000',
  borderRightWidth: 1,
  borderRightColor: '#000',
  shadowColor: '#000',
  shadowOffset: { width: 0, height: 2 },
  shadowOpacity: 0.25,
  shadowRadius: 3.84,
  elevation: 5, // Android shadow
}
```

### Text Properties

```tsx
{
  color: '#000',
  fontSize: 16,
  fontWeight: 'bold', // 'normal' | 'bold' | '100' | '200' | ... | '900'
  fontStyle: 'normal', // 'normal' | 'italic'
  fontFamily: 'System',
  textAlign: 'center', // 'auto' | 'left' | 'right' | 'center' | 'justify'
  textDecorationLine: 'underline', // 'none' | 'underline' | 'line-through' | 'underline line-through'
  textDecorationColor: '#000',
  textDecorationStyle: 'solid', // 'solid' | 'double' | 'dotted' | 'dashed'
  textTransform: 'uppercase', // 'none' | 'uppercase' | 'lowercase' | 'capitalize'
  letterSpacing: 1,
  lineHeight: 20,
}
```

## Platform-Specific Styles

Use `Platform.select()` for platform-specific styles.

```tsx
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    ...Platform.select({
      ios: {
        paddingTop: 20,
      },
      android: {
        paddingTop: 10,
      },
      default: {
        paddingTop: 15,
      },
    }),
  },
});
```

Or use platform-specific style files:

```tsx
// styles.ios.js
export default StyleSheet.create({ /* iOS styles */ });

// styles.android.js
export default StyleSheet.create({ /* Android styles */ });

// styles.js
import { Platform } from 'react-native';
const styles = Platform.OS === 'ios' 
  ? require('./styles.ios')
  : require('./styles.android');
```

## Dynamic Styles

For truly dynamic styles, use inline objects or functions.

```tsx
const App = ({ color, size }) => {
  const dynamicStyle = {
    backgroundColor: color,
    width: size,
    height: size,
  };
  
  return <View style={[styles.base, dynamicStyle]} />;
};

// Or with a function
const getButtonStyle = (isActive) => ({
  ...styles.button,
  backgroundColor: isActive ? 'blue' : 'gray',
});

<View style={getButtonStyle(true)} />
```

## StyleSheet.flatten()

Flattens a style array into a single object (useful for debugging).

```tsx
const flattened = StyleSheet.flatten([styles.base, styles.highlighted]);
console.log(flattened); // Single object with merged styles
```

## Best Practices

1. **Always use StyleSheet.create()** - Better performance than inline objects
2. **Compose styles with arrays** - `[styles.base, styles.variant]`
3. **Keep styles close to components** - Co-locate styles with components
4. **Use descriptive names** - `container`, `title`, `button` not `s1`, `s2`
5. **Extract common styles** - Create shared style modules for reuse
6. **Use Platform.select()** - For platform-specific styling needs
7. **Avoid inline objects** - Only for truly dynamic values
8. **Group related styles** - Keep layout, appearance, and text styles organized

## Performance Tips

- **StyleSheet.create()** creates styles once, not on every render
- **Style arrays** are optimized for reference equality checks
- **Avoid creating styles in render** - Move to StyleSheet.create() or component level
- **Use flatten() sparingly** - Only for debugging, not production code

<!--
Source references:
- https://reactnative.dev/docs/stylesheet
- https://reactnative.dev/docs/style
- https://reactnative.dev/docs/view-style-props
- https://reactnative.dev/docs/text-style-props
-->
