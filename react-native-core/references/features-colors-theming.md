---
name: react-native-colors-theming
description: Colors, PlatformColor, Appearance API for theming and dark mode. Use when working with colors, implementing dark mode, or accessing platform-specific colors.
---

# Colors and Theming

React Native provides flexible color APIs for working with colors, platform-specific colors, and appearance (dark mode) support.

## Color Representations

React Native supports multiple color formats.

### Hexadecimal

```tsx
'#f0f'           // #rgb
'#ff00ff'        // #rrggbb
'#f0ff'          // #rgba
'#ff00ff00'      // #rrggbbaa
```

### RGB/RGBA

```tsx
'rgb(255, 0, 255)'
'rgb(255 0 255)'
'rgba(255, 0, 255, 1.0)'
'rgba(255 0 255 / 1.0)'
```

### HSL/HSLA

```tsx
'hsl(360, 100%, 100%)'
'hsl(360 100% 100%)'
'hsla(360, 100%, 100%, 1.0)'
'hsla(360 100% 100% / 1.0)'
```

### HWB

```tsx
'hwb(0, 0%, 100%)'
'hwb(360 100% 100%)'
```

### Color Ints

```tsx
0xff00ff00  // 0xrrggbbaa (RGB mode)
```

### Named Colors

React Native supports CSS3/SVG named colors (lowercase only):

```tsx
'transparent'
'aliceblue'
'aqua'
'black'
'blue'
'cyan'
'gray'
'green'
'red'
'white'
// ... and many more
```

## PlatformColor

Access platform-specific color system colors.

### iOS

```tsx
import { PlatformColor } from 'react-native';

const styles = StyleSheet.create({
  text: {
    color: PlatformColor('labelColor'),           // Text color
    backgroundColor: PlatformColor('systemBackgroundColor'),
  },
  button: {
    backgroundColor: PlatformColor('systemBlue'),
  },
});
```

**Common iOS Colors:**
- `labelColor` - Primary text color
- `secondaryLabelColor` - Secondary text color
- `tertiaryLabelColor` - Tertiary text color
- `systemBackgroundColor` - Background color
- `secondarySystemBackgroundColor` - Secondary background
- `systemBlue`, `systemGreen`, `systemRed`, etc. - System colors

### Android

```tsx
import { PlatformColor } from 'react-native';

const styles = StyleSheet.create({
  text: {
    color: PlatformColor('?attr/colorPrimary'),
    backgroundColor: PlatformColor('?android:attr/colorBackground'),
  },
});
```

**Common Android Colors:**
- `?attr/colorPrimary` - Primary color
- `?attr/colorPrimaryDark` - Dark primary color
- `?android:attr/colorBackground` - Background color
- `?android:attr/textColorPrimary` - Primary text color

### Cross-Platform

```tsx
import { Platform, PlatformColor } from 'react-native';

const primaryColor = Platform.select({
  ios: PlatformColor('systemBlue'),
  android: PlatformColor('?attr/colorPrimary'),
  default: '#007AFF',
});
```

## DynamicColorIOS

iOS-specific API for light/dark mode colors.

```tsx
import { DynamicColorIOS } from 'react-native';

const styles = StyleSheet.create({
  container: {
    backgroundColor: DynamicColorIOS({
      light: '#ffffff',
      dark: '#000000',
    }),
  },
  text: {
    color: DynamicColorIOS({
      light: '#000000',
      dark: '#ffffff',
    }),
  },
});
```

## Appearance API

Detect and respond to appearance changes (light/dark mode).

```tsx
import { Appearance, useColorScheme } from 'react-native';

// Hook (recommended)
const App = () => {
  const colorScheme = useColorScheme();
  const isDark = colorScheme === 'dark';

  return (
    <View style={{
      backgroundColor: isDark ? '#000' : '#fff',
      color: isDark ? '#fff' : '#000',
    }}>
      {/* Content */}
    </View>
  );
};

// Direct API
const colorScheme = Appearance.getColorScheme(); // 'light' | 'dark' | null

// Listen to changes
useEffect(() => {
  const subscription = Appearance.addChangeListener(({ colorScheme }) => {
    console.log('Appearance changed:', colorScheme);
  });

  return () => subscription.remove();
}, []);
```

### useColorScheme Hook

```tsx
import { useColorScheme } from 'react-native';

const App = () => {
  const colorScheme = useColorScheme(); // 'light' | 'dark' | null

  const styles = StyleSheet.create({
    container: {
      backgroundColor: colorScheme === 'dark' ? '#000' : '#fff',
    },
  });

  return <View style={styles.container} />;
};
```

## Theming Pattern

Create a theme system with colors and appearance support.

```tsx
import { useColorScheme } from 'react-native';

const lightTheme = {
  background: '#ffffff',
  text: '#000000',
  primary: '#007AFF',
};

const darkTheme = {
  background: '#000000',
  text: '#ffffff',
  primary: '#0A84FF',
};

const useTheme = () => {
  const colorScheme = useColorScheme();
  return colorScheme === 'dark' ? darkTheme : lightTheme;
};

const App = () => {
  const theme = useTheme();

  return (
    <View style={{ backgroundColor: theme.background }}>
      <Text style={{ color: theme.text }}>Hello</Text>
    </View>
  );
};
```

## Best Practices

1. **Use PlatformColor** for platform-specific colors when possible
2. **Support dark mode** with Appearance API or DynamicColorIOS
3. **Test both themes** - Ensure readability in light and dark modes
4. **Use semantic colors** - `labelColor` instead of hardcoded `#000`
5. **Create theme objects** - Centralize color definitions
6. **Use useColorScheme hook** - Prefer over direct Appearance API
7. **Provide fallbacks** - Handle null colorScheme gracefully

## Common Patterns

### Theme Hook

```tsx
const useAppTheme = () => {
  const colorScheme = useColorScheme();
  
  return {
    isDark: colorScheme === 'dark',
    colors: colorScheme === 'dark' ? darkColors : lightColors,
    spacing: { small: 8, medium: 16, large: 24 },
  };
};
```

### Platform-Specific Colors

```tsx
const getPrimaryColor = () => {
  if (Platform.OS === 'ios') {
    return PlatformColor('systemBlue');
  } else if (Platform.OS === 'android') {
    return PlatformColor('?attr/colorPrimary');
  }
  return '#007AFF';
};
```

### Dynamic Styling

```tsx
const styles = (isDark: boolean) => StyleSheet.create({
  container: {
    backgroundColor: isDark ? '#000' : '#fff',
  },
});

// Usage
const App = () => {
  const isDark = useColorScheme() === 'dark';
  return <View style={styles(isDark).container} />;
};
```

<!--
Source references:
- https://reactnative.dev/docs/colors
- https://reactnative.dev/docs/platformcolor
- https://reactnative.dev/docs/appearance
- https://reactnative.dev/docs/dynamiccolorios
-->
