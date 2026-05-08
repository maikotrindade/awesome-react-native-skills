---
name: react-native-platform-apis
description: Platform APIs - Linking, Dimensions, Platform detection, AppState, and other native integrations. Use when accessing device capabilities, handling deep links, or detecting platform differences.
---

# Platform APIs

React Native provides APIs to access native platform features and handle platform-specific behavior.

## Platform Detection

Detect the current platform and apply platform-specific code.

```tsx
import { Platform } from 'react-native';

// Platform OS
if (Platform.OS === 'ios') {
  // iOS-specific code
} else if (Platform.OS === 'android') {
  // Android-specific code
}

// Platform version
if (Platform.Version >= 13) {
  // iOS 13+ or Android API 13+
}

// Platform select for values
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

// Platform select for components
const Component = Platform.select({
  ios: () => require('./ComponentIOS'),
  android: () => require('./ComponentAndroid'),
})();
```

## Dimensions

Get device screen dimensions and respond to changes.

```tsx
import { Dimensions } from 'react-native';

// Get initial dimensions
const { width, height } = Dimensions.get('window');
// or
const { width, height } = Dimensions.get('screen');

// Subscribe to dimension changes
const subscription = Dimensions.addEventListener('change', ({ window, screen }) => {
  console.log('Window:', window.width, window.height);
  console.log('Screen:', screen.width, screen.height);
});

// Cleanup
subscription?.remove();
```

### Responsive Layout

```tsx
import { useWindowDimensions } from 'react-native';

const App = () => {
  const { width, height } = useWindowDimensions();
  const isTablet = width >= 768;

  return (
    <View style={isTablet ? styles.tabletLayout : styles.phoneLayout}>
      {/* Content */}
    </View>
  );
};
```

## Linking

Handle deep links and universal links.

### Opening URLs

```tsx
import { Linking } from 'react-native';

// Open URL
Linking.openURL('https://example.com');

// Check if URL can be opened
const canOpen = await Linking.canOpenURL('https://example.com');

// Open phone dialer
Linking.openURL('tel:+1234567890');

// Open email client
Linking.openURL('mailto:support@example.com');

// Open SMS
Linking.openURL('sms:+1234567890');
```

### Handling Deep Links

```tsx
import { useEffect } from 'react';
import { Linking } from 'react-native';

const App = () => {
  useEffect(() => {
    // Get initial URL if app was opened via link
    Linking.getInitialURL().then(url => {
      if (url) {
        handleDeepLink(url);
      }
    });

    // Listen for incoming links
    const subscription = Linking.addEventListener('url', ({ url }) => {
      handleDeepLink(url);
    });

    return () => {
      subscription.remove();
    };
  }, []);

  const handleDeepLink = (url) => {
    // Parse URL and navigate
    console.log('Deep link:', url);
  };
};
```

### Parsing URLs

```tsx
import { Linking } from 'react-native';

const url = 'myapp://product/123?color=red';
const parsed = Linking.parse(url);

console.log(parsed.scheme);  // 'myapp'
console.log(parsed.hostname); // 'product'
console.log(parsed.path);    // '/123'
console.log(parsed.queryParams); // { color: 'red' }
```

## AppState

Track application state (active, background, inactive).

```tsx
import { useEffect, useRef } from 'react';
import { AppState } from 'react-native';

const App = () => {
  const appState = useRef(AppState.currentState);

  useEffect(() => {
    const subscription = AppState.addEventListener('change', nextAppState => {
      if (
        appState.current.match(/inactive|background/) &&
        nextAppState === 'active'
      ) {
        console.log('App has come to the foreground!');
      }

      if (nextAppState.match(/inactive|background/)) {
        console.log('App has gone to the background!');
      }

      appState.current = nextAppState;
    });

    return () => {
      subscription?.remove();
    };
  }, []);

  return null;
};
```

### AppState Values

- `active` - App is running in foreground
- `background` - App is running in background
- `inactive` - Transitioning between states (iOS only)

## Clipboard

Copy and paste text to/from clipboard.

```tsx
import Clipboard from '@react-native-clipboard/clipboard';

// Copy to clipboard
Clipboard.setString('Hello, World!');

// Get from clipboard
const text = await Clipboard.getString();

// Check if clipboard has content
const hasString = await Clipboard.hasString();
```

## Keyboard

Control keyboard behavior and listen to keyboard events.

```tsx
import { Keyboard } from 'react-native';

// Dismiss keyboard
Keyboard.dismiss();

// Listen to keyboard events
useEffect(() => {
  const showSubscription = Keyboard.addListener('keyboardDidShow', (e) => {
    console.log('Keyboard height:', e.endCoordinates.height);
  });

  const hideSubscription = Keyboard.addListener('keyboardDidHide', () => {
    console.log('Keyboard hidden');
  });

  return () => {
    showSubscription.remove();
    hideSubscription.remove();
  };
}, []);
```

### KeyboardAvoidingView

Automatically adjust view when keyboard appears.

```tsx
import { KeyboardAvoidingView, Platform } from 'react-native';

<KeyboardAvoidingView
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
  style={{ flex: 1 }}
>
  <TextInput />
</KeyboardAvoidingView>
```

**Behavior values:**
- `padding` - Add padding when keyboard appears (iOS)
- `height` - Resize view height (Android)
- `position` - Move view position

## Alert

Show native alert dialogs.

```tsx
import { Alert } from 'react-native';

// Simple alert
Alert.alert('Title', 'Message');

// Alert with buttons
Alert.alert(
  'Title',
  'Message',
  [
    {
      text: 'Cancel',
      onPress: () => console.log('Cancel'),
      style: 'cancel',
    },
    {
      text: 'OK',
      onPress: () => console.log('OK'),
    },
  ]
);

// Alert with three buttons
Alert.alert(
  'Title',
  'Message',
  [
    { text: 'Cancel', style: 'cancel' },
    { text: 'Delete', style: 'destructive', onPress: () => {} },
    { text: 'OK', onPress: () => {} },
  ],
  { cancelable: false }
);
```

## Permissions

Request platform permissions (requires additional libraries).

```tsx
// Using react-native-permissions
import { request, PERMISSIONS, RESULTS } from 'react-native-permissions';

const requestCameraPermission = async () => {
  const result = await request(
    Platform.OS === 'ios'
      ? PERMISSIONS.IOS.CAMERA
      : PERMISSIONS.ANDROID.CAMERA
  );

  if (result === RESULTS.GRANTED) {
    console.log('Permission granted');
  }
};
```

## PixelRatio

Access device pixel density.

```tsx
import { PixelRatio } from 'react-native';

const ratio = PixelRatio.get(); // e.g., 2, 3 for Retina displays

// Convert pixels to density-independent pixels
const dp = PixelRatio.getPixelSizeForLayoutSize(100);

// Get font scale (user's accessibility setting)
const fontScale = PixelRatio.getFontScale();
```

## StatusBar

Control status bar appearance.

```tsx
import { StatusBar } from 'react-native';

<StatusBar
  barStyle="dark-content" // 'default' | 'light-content' | 'dark-content'
  hidden={false}
  backgroundColor="#fff" // Android only
  translucent={true}     // Android only
/>
```

## BackHandler (Android)

Handle Android hardware back button presses.

```tsx
import { useEffect } from 'react';
import { BackHandler } from 'react-native';

const App = () => {
  useEffect(() => {
    const backAction = () => {
      // Handle back button
      if (shouldPreventBack) {
        return true; // Prevent default back action
      }
      return false; // Allow default back action
    };

    const backHandler = BackHandler.addEventListener(
      'hardwareBackPress',
      backAction
    );

    return () => backHandler.remove();
  }, []);

  return <View />;
};
```

### BackHandler Methods

```tsx
// Add listener
const subscription = BackHandler.addEventListener(
  'hardwareBackPress',
  () => {
    // Return true to prevent default, false to allow
    return true;
  }
);

// Remove listener
subscription.remove();

// Exit app (Android only)
BackHandler.exitApp();
```

**Note**: Event subscriptions called in reverse order. If one returns `true`, earlier subscriptions won't be called.

## I18nManager

Manage Right-to-Left (RTL) layout support for languages like Arabic and Hebrew.

```tsx
import { I18nManager } from 'react-native';

// Check if RTL is enabled
const isRTL = I18nManager.isRTL;

// Force RTL (development only, requires reload)
I18nManager.forceRTL(true);

// Allow RTL (development only)
I18nManager.allowRTL(true);

// Swap left/right values
I18nManager.swapLeftAndRightInRTL(true);
```

### RTL-Aware Layout

```tsx
import { I18nManager } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flexDirection: I18nManager.isRTL ? 'row-reverse' : 'row',
  },
  button: {
    marginLeft: I18nManager.isRTL ? 0 : 10,
    marginRight: I18nManager.isRTL ? 10 : 0,
  },
});
```

### RTL-Aware Positioning

```tsx
<View style={{
  position: 'absolute',
  left: I18nManager.isRTL ? undefined : 0,
  right: I18nManager.isRTL ? 0 : undefined,
}}>
  {I18nManager.isRTL ? <Text>Back →</Text> : <Text>← Back</Text>}
</View>
```

## Best Practices

1. **Use Platform.select()** for platform-specific code
2. **Handle dimension changes** - Subscribe to dimension events for responsive layouts
3. **Test deep links** on both platforms - behavior differs
4. **Clean up listeners** - Remove event listeners in useEffect cleanup
5. **Check permissions** before accessing device features
6. **Use KeyboardAvoidingView** for forms to prevent keyboard overlap
7. **Handle AppState changes** - Pause/resume timers, network requests
8. **Test on real devices** - Simulator behavior may differ

## Common Patterns

### Platform-Specific Component

```tsx
// Component.ios.jsx
export default function Component() {
  return <View style={styles.iosStyle} />;
}

// Component.android.jsx
export default function Component() {
  return <View style={styles.androidStyle} />;
}

// Component.js
import Component from './Component';
export default Component;
```

### Responsive Hook

```tsx
import { useState, useEffect } from 'react';
import { Dimensions } from 'react-native';

const useWindowDimensions = () => {
  const [dimensions, setDimensions] = useState(Dimensions.get('window'));

  useEffect(() => {
    const subscription = Dimensions.addEventListener('change', ({ window }) => {
      setDimensions(window);
    });

    return () => subscription?.remove();
  }, []);

  return dimensions;
};
```

### Deep Link Handler

```tsx
const useDeepLinking = () => {
  useEffect(() => {
    const handleURL = (url) => {
      const route = url.replace(/.*?:\/\//g, '');
      // Navigate based on route
    };

    Linking.getInitialURL().then(url => url && handleURL(url));
    const subscription = Linking.addEventListener('url', ({ url }) => handleURL(url));

    return () => subscription.remove();
  }, []);
};
```

<!--
Source references:
- https://reactnative.dev/docs/platform-specific-code
- https://reactnative.dev/docs/dimensions
- https://reactnative.dev/docs/linking
- https://reactnative.dev/docs/appstate
- https://reactnative.dev/docs/keyboardavoidingview
- https://reactnative.dev/docs/alert
- https://reactnative.dev/docs/statusbar
-->
