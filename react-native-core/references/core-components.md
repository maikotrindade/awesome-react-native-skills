---
name: react-native-core-components
description: Core React Native components - View, Text, Image, TextInput, ScrollView. Use when building UI layouts, displaying content, handling user input, or creating scrollable content.
---

# Core Components

React Native provides essential Core Components that map to native platform views. These are the building blocks for creating mobile UIs.

## View

The most fundamental component for building UI. A container that supports flexbox layout, styling, touch handling, and accessibility controls.

```tsx
import { View, Text } from 'react-native';

const App = () => (
  <View style={{ flexDirection: 'row', padding: 10 }}>
    <View style={{ height: 100, backgroundColor: 'blue', flex: 0.2 }} />
    <View style={{ height: 100, backgroundColor: 'red', flex: 0.4 }} />
    <Text>Hello World!</Text>
  </View>
);
```

### Key Features

- **Layout**: Supports flexbox for positioning children
- **Styling**: Accepts style prop (use StyleSheet for performance)
- **Touch handling**: Responder props like `onResponderMove`
- **Accessibility**: Full accessibility API support
- **Nesting**: Can contain 0 to many children of any type

### Common Props

```tsx
<View
  style={styles.container}
  onLayout={(event) => {
    const { width, height } = event.nativeEvent.layout;
  }}
  pointerEvents="auto" // 'none' | 'box-none' | 'box-only' | 'auto'
  removeClippedSubviews={true} // Performance optimization
>
  {/* children */}
</View>
```

## Text

Component for displaying text. Supports nesting, styling, and touch handling.

```tsx
import { Text, StyleSheet } from 'react-native';

const App = () => (
  <Text style={styles.baseText}>
    <Text style={styles.titleText} onPress={() => console.log('pressed')}>
      Title
      {'\n\n'}
    </Text>
    <Text numberOfLines={5}>Body text content</Text>
  </Text>
);
```

### Key Features

- **Nesting**: Can nest Text components for formatted text
- **Styling**: Inherits styles from parent Text components
- **Touch events**: `onPress`, `onLongPress` handlers
- **Text selection**: `selectable` prop for copy/paste

### Common Props

```tsx
<Text
  style={styles.text}
  numberOfLines={2} // Truncate after N lines
  ellipsizeMode="tail" // 'head' | 'middle' | 'tail' | 'clip'
  selectable={true}
  onPress={() => {}}
  onLongPress={() => {}}
>
  Content
</Text>
```

## Image

Component for displaying images from various sources: network, static resources, local files, or data URIs.

```tsx
import { Image, StyleSheet } from 'react-native';

const App = () => (
  <>
    {/* Local image */}
    <Image
      style={styles.logo}
      source={require('./assets/logo.png')}
    />
    
    {/* Network image */}
    <Image
      style={styles.logo}
      source={{
        uri: 'https://example.com/image.png',
      }}
    />
    
    {/* Data URI */}
    <Image
      style={styles.logo}
      source={{
        uri: 'data:image/png;base64,iVBORw0KG...',
      }}
    />
  </>
);
```

### Key Features

- **Multiple sources**: require(), URI objects, data URIs
- **Resize modes**: `cover`, `contain`, `stretch`, `repeat`, `center`
- **Loading states**: `onLoad`, `onLoadStart`, `onLoadEnd`, `onError`
- **Caching**: Automatic caching for network images

### Common Props

```tsx
<Image
  source={require('./image.png')}
  style={styles.image}
  resizeMode="cover" // 'cover' | 'contain' | 'stretch' | 'repeat' | 'center'
  onLoad={() => console.log('loaded')}
  onError={(error) => console.log('error', error)}
  defaultSource={require('./placeholder.png')} // iOS only
  loadingIndicatorSource={require('./loading.png')} // Android only
/>
```

**Note**: Network and data images require explicit dimensions.

## ImageBackground

Component for displaying an image as a background with children layered on top.

```tsx
import { ImageBackground, Text } from 'react-native';

const App = () => (
  <ImageBackground
    source={require('./background.png')}
    resizeMode="cover"
    style={styles.background}
  >
    <Text style={styles.text}>Content on top</Text>
  </ImageBackground>
);
```

### ImageBackground Props

```tsx
<ImageBackground
  source={require('./image.png')}
  style={styles.container}
  imageStyle={styles.image}  // Styles for the image itself
  imageRef={(ref) => {}}     // Ref to inner Image component
>
  {/* Children rendered on top */}
</ImageBackground>
```

**Note**: Must specify width and height in style. Inherits all Image props.

## TextInput

Component for text input via keyboard. Supports various keyboard types, auto-correction, and validation.

```tsx
import { useState } from 'react';
import { TextInput, StyleSheet } from 'react-native';

const App = () => {
  const [text, setText] = useState('');
  
  return (
    <TextInput
      style={styles.input}
      onChangeText={setText}
      value={text}
      placeholder="Enter text"
      keyboardType="default" // 'default' | 'numeric' | 'email-address' | etc.
      autoCapitalize="sentences" // 'none' | 'sentences' | 'words' | 'characters'
      autoCorrect={true}
    />
  );
};
```

### Key Features

- **Controlled component**: Use `value` and `onChangeText` for controlled input
- **Keyboard types**: `numeric`, `email-address`, `phone-pad`, `url`, etc.
- **Multiline**: Set `multiline={true}` for text areas
- **Focus control**: `.focus()` and `.blur()` methods

### Common Props

```tsx
<TextInput
  value={text}
  onChangeText={setText}
  placeholder="Placeholder text"
  keyboardType="email-address"
  autoCapitalize="none"
  autoCorrect={false}
  secureTextEntry={true} // For passwords
  multiline={false}
  numberOfLines={4} // For multiline
  maxLength={100}
  editable={true}
  onFocus={() => {}}
  onBlur={() => {}}
  onSubmitEditing={() => {}}
/>
```

### Multiline Considerations

- Border styles (e.g., `borderBottomColor`) don't apply when `multiline=true`
- Wrap in View for custom borders on multiline inputs
- `numberOfLines` sets initial height but doesn't limit input

## ScrollView

Scrollable container component. Renders all children at once (use FlatList for long lists).

```tsx
import { ScrollView, Text, StyleSheet } from 'react-native';

const App = () => (
  <ScrollView
    style={styles.scrollView}
    contentContainerStyle={styles.content}
    showsVerticalScrollIndicator={true}
  >
    <Text>Content item 1</Text>
    <Text>Content item 2</Text>
    {/* More content */}
  </ScrollView>
);
```

### Key Features

- **Bounded height required**: Must have explicit or flex height
- **All children rendered**: Not optimized for long lists
- **Scroll events**: `onScroll`, `scrollEventThrottle` for performance
- **Refresh control**: Use `refreshControl` prop for pull-to-refresh

### Common Props

```tsx
<ScrollView
  style={styles.container}
  contentContainerStyle={styles.content} // Styles for content wrapper
  horizontal={false}
  showsVerticalScrollIndicator={true}
  showsHorizontalScrollIndicator={true}
  onScroll={(event) => {
    const offsetY = event.nativeEvent.contentOffset.y;
  }}
  scrollEventThrottle={16} // Throttle scroll events (ms)
  bounces={true} // iOS bounce effect
  scrollEnabled={true}
  pagingEnabled={false} // Snap to pages
  decelerationRate="normal" // 'normal' | 'fast'
/>
```

### When to Use ScrollView vs FlatList

- **ScrollView**: Small, fixed content; all items visible; simple layout
- **FlatList**: Long lists; dynamic content; performance-critical; need separators/headers

## Best Practices

1. **Use StyleSheet.create()** for styles instead of inline objects
2. **Set explicit dimensions** for network images
3. **Use FlatList** instead of ScrollView for long lists
4. **Control TextInput** with `value` and `onChangeText` for React state
5. **Handle keyboard** with KeyboardAvoidingView for forms
6. **Optimize ScrollView** with `scrollEventThrottle` and `removeClippedSubviews`

<!--
Source references:
- https://reactnative.dev/docs/view
- https://reactnative.dev/docs/text
- https://reactnative.dev/docs/image
- https://reactnative.dev/docs/textinput
- https://reactnative.dev/docs/scrollview
- https://reactnative.dev/docs/intro-react-native-components
-->
