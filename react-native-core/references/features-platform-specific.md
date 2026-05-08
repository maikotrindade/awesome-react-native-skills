---
name: react-native-platform-specific
description: Platform-specific APIs — Android (ToastAndroid, DrawerLayoutAndroid, PermissionsAndroid) and iOS (ActionSheetIOS). Use when implementing platform-native UI patterns.
---

# Platform-Specific Components and APIs

React Native provides platform-specific components and APIs for Android and iOS native features.

> **Permissions note**: `PermissionsAndroid` is the low-level Android API. For production apps, use [`react-native-permissions`](https://github.com/zoontek/react-native-permissions) — it provides a unified API for both iOS and Android, handles `blocked` status, and works with Expo. See the `react-native-ecosystem` skill for the full permission request flow.

## Android APIs

### PermissionsAndroid (low-level)

Request Android runtime permissions (API 23+). Prefer `react-native-permissions` for cross-platform apps.

```tsx
import { PermissionsAndroid, Platform } from 'react-native';

const requestCameraPermission = async (): Promise<boolean> => {
  if (Platform.OS !== 'android') return true;

  try {
    const granted = await PermissionsAndroid.request(
      PermissionsAndroid.PERMISSIONS.CAMERA,
      {
        title: 'Camera Permission',
        message: 'App needs access to your camera',
        buttonNeutral: 'Ask Me Later',
        buttonNegative: 'Cancel',
        buttonPositive: 'OK',
      }
    );
    return granted === PermissionsAndroid.RESULTS.GRANTED;
  } catch (err) {
    return false;
  }
};
```

#### Available Permissions

```tsx
PermissionsAndroid.PERMISSIONS.CAMERA
PermissionsAndroid.PERMISSIONS.RECORD_AUDIO
PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION
PermissionsAndroid.PERMISSIONS.ACCESS_COARSE_LOCATION
PermissionsAndroid.PERMISSIONS.READ_EXTERNAL_STORAGE    // < Android 13
PermissionsAndroid.PERMISSIONS.WRITE_EXTERNAL_STORAGE
PermissionsAndroid.PERMISSIONS.READ_CONTACTS
PermissionsAndroid.PERMISSIONS.READ_MEDIA_IMAGES        // Android 13+
PermissionsAndroid.PERMISSIONS.READ_MEDIA_VIDEO         // Android 13+
PermissionsAndroid.PERMISSIONS.POST_NOTIFICATIONS       // Android 13+
```

#### Request Multiple Permissions

```tsx
const granted = await PermissionsAndroid.requestMultiple([
  PermissionsAndroid.PERMISSIONS.CAMERA,
  PermissionsAndroid.PERMISSIONS.RECORD_AUDIO,
]);

const cameraOk = granted[PermissionsAndroid.PERMISSIONS.CAMERA] === PermissionsAndroid.RESULTS.GRANTED;
const audioOk = granted[PermissionsAndroid.PERMISSIONS.RECORD_AUDIO] === PermissionsAndroid.RESULTS.GRANTED;
```

### ToastAndroid

Shows a brief native Android toast notification.

```tsx
import { ToastAndroid, Platform } from 'react-native';

// Only works on Android
if (Platform.OS === 'android') {
  ToastAndroid.show('Saved!', ToastAndroid.SHORT);
  // ToastAndroid.SHORT = ~2 seconds
  // ToastAndroid.LONG = ~3.5 seconds
}

// With gravity
ToastAndroid.showWithGravity(
  'Item deleted',
  ToastAndroid.SHORT,
  ToastAndroid.BOTTOM  // TOP | CENTER | BOTTOM
);
```

> On iOS, use a custom in-app toast or the `Alert` API — no native toast equivalent exists.

### DrawerLayoutAndroid

Native Android drawer navigation (side menu). For production use React Navigation's Drawer navigator instead.

```tsx
import { DrawerLayoutAndroid, View, Text, Pressable } from 'react-native';
import { useRef } from 'react';

const App = () => {
  const drawer = useRef<DrawerLayoutAndroid>(null);

  const navigationView = () => (
    <View style={{ flex: 1, padding: 24 }}>
      <Text>Menu Item 1</Text>
      <Text>Menu Item 2</Text>
    </View>
  );

  return (
    <DrawerLayoutAndroid
      ref={drawer}
      drawerWidth={300}
      drawerPosition="left"
      renderNavigationView={navigationView}
    >
      <Pressable onPress={() => drawer.current?.openDrawer()}>
        <Text>Open Drawer</Text>
      </Pressable>
    </DrawerLayoutAndroid>
  );
};
```

---

## iOS APIs

### ActionSheetIOS

Shows native iOS action sheet or share sheet.

```tsx
import { ActionSheetIOS, Platform } from 'react-native';

const showOptions = () => {
  if (Platform.OS !== 'ios') return;

  ActionSheetIOS.showActionSheetWithOptions(
    {
      options: ['Cancel', 'Edit', 'Delete'],
      cancelButtonIndex: 0,
      destructiveButtonIndex: 2,
      title: 'Post Options',
      message: 'Choose an action',
    },
    (buttonIndex) => {
      if (buttonIndex === 1) handleEdit();
      if (buttonIndex === 2) handleDelete();
    }
  );
};
```

#### Share Sheet (iOS)

```tsx
ActionSheetIOS.showShareActionSheetWithOptions(
  {
    url: 'https://example.com',
    message: 'Check this out',
    subject: 'Interesting link',  // email subject
    excludedActivityTypes: [
      'com.apple.UIKit.activity.PostToFacebook',
    ],
  },
  (error) => console.error(error),
  (completed, method) => {
    if (completed) console.log('Shared via:', method);
  }
);
```

---

## Cross-Platform Patterns

Use `Platform.select()` or file extensions for platform-specific implementations:

```tsx
// Inline
const message = Platform.select({
  ios: 'Tap to continue',
  android: 'Touch to continue',
  default: 'Click to continue',
});

// Component files
// Button.ios.tsx  — auto-used on iOS
// Button.android.tsx — auto-used on Android
// Button.tsx — fallback for web/other
```
