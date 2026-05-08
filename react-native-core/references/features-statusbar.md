---
name: react-native-statusbar
description: StatusBar component for controlling app status bar appearance. Use when customizing status bar style, visibility, or animations.
---

# StatusBar

Component to control the app's status bar appearance and behavior.

## Basic Usage

```tsx
import { StatusBar } from 'react-native';

const App = () => (
  <>
    <StatusBar barStyle="dark-content" />
    {/* App content */}
  </>
);
```

## StatusBar Styles

### barStyle

Controls status bar content color.

```tsx
<StatusBar barStyle="dark-content" />  // Dark text/icons
<StatusBar barStyle="light-content" /> // Light text/icons
<StatusBar barStyle="default" />      // Platform default
```

**Platform behavior:**
- **iOS**: Controls text/icons color
- **Android**: Controls icons color (API 23+)

### backgroundColor (Android)

Sets status bar background color.

```tsx
<StatusBar
  backgroundColor="#61dafb"
  barStyle="dark-content"
/>
```

**Note**: Only works on Android API 21+.

### translucent (Android)

Makes status bar translucent (content can draw behind it).

```tsx
<StatusBar translucent={true} />
```

## Visibility

### hidden

Hide or show the status bar.

```tsx
<StatusBar hidden={true} />
```

### showHideTransition (iOS)

Animation when showing/hiding status bar.

```tsx
<StatusBar
  hidden={hidden}
  showHideTransition="fade"  // 'fade' | 'slide' | 'none'
/>
```

## Animations

### animated

Enable animated transitions for style changes.

```tsx
<StatusBar
  animated={true}
  barStyle={isDark ? 'light-content' : 'dark-content'}
/>
```

## Multiple StatusBar Components

Multiple `StatusBar` components can be mounted. Props merge in mount order.

```tsx
// In navigation header
<StatusBar barStyle="light-content" />

// In specific screen
<StatusBar barStyle="dark-content" />
```

## Common Patterns

### Dynamic Style Based on Theme

```tsx
import { useColorScheme } from 'react-native';

const App = () => {
  const colorScheme = useColorScheme();
  
  return (
    <>
      <StatusBar
        barStyle={colorScheme === 'dark' ? 'light-content' : 'dark-content'}
        animated={true}
      />
      {/* Content */}
    </>
  );
};
```

### Per-Screen StatusBar

```tsx
const Screen = () => (
  <>
    <StatusBar barStyle="light-content" backgroundColor="#000" />
    <View style={{ backgroundColor: '#000' }}>
      {/* Content */}
    </View>
  </>
);
```

### Hide StatusBar for Full Screen

```tsx
const FullScreenView = () => (
  <>
    <StatusBar hidden={true} />
    <View style={{ flex: 1 }}>
      {/* Full screen content */}
    </View>
  </>
);
```

## Best Practices

1. **Set barStyle** - Match your app's design
2. **Use animated** - Smooth transitions between styles
3. **Test on both platforms** - Behavior differs
4. **Consider safe areas** - StatusBar affects layout
5. **Match background** - Coordinate with app background color
6. **Handle theme changes** - Update style when theme changes

## Platform-Specific Notes

### iOS

- `barStyle` controls text/icons color
- `showHideTransition` for show/hide animations
- No `backgroundColor` prop

### Android

- `barStyle` controls icons color (API 23+)
- `backgroundColor` sets background color (API 21+)
- `translucent` allows content behind status bar

<!--
Source references:
- https://reactnative.dev/docs/statusbar
-->
