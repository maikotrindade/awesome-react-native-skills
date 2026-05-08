# Features — Permissions

## Libraries

- **`react-native-permissions`** — unified API for bare React Native and Expo bare workflow
- **`expo-*` modules** — permission APIs bundled per-feature (camera, location, etc.) for Expo managed workflow

---

## react-native-permissions

### Installation
```bash
npm install react-native-permissions
```

Configure in `Info.plist` (iOS) and `AndroidManifest.xml` for each permission you use.

### Permission Statuses

| Status | Meaning |
|--------|---------|
| `unavailable` | Feature not available on this device |
| `denied` | Not requested yet, or denied (can request again) |
| `limited` | Granted with limitations (iOS photo library) |
| `granted` | Fully granted |
| `blocked` | User denied and selected "Don't ask again" — must open Settings |

### Check a Permission
```ts
import { check, PERMISSIONS, type PermissionStatus } from 'react-native-permissions';
import { Platform } from 'react-native';

const cameraPermission = Platform.select({
  ios: PERMISSIONS.IOS.CAMERA,
  android: PERMISSIONS.ANDROID.CAMERA,
})!;

const status: PermissionStatus = await check(cameraPermission);
```

### Request a Permission
```ts
import { request, PERMISSIONS } from 'react-native-permissions';

const status = await request(PERMISSIONS.IOS.CAMERA, {
  title: 'Camera Access',
  message: 'This app needs camera access to scan QR codes.',
  buttonPositive: 'Allow',
  buttonNegative: 'Deny',
});
```

### Request Multiple Permissions
```ts
import { requestMultiple, PERMISSIONS } from 'react-native-permissions';

const statuses = await requestMultiple([
  PERMISSIONS.IOS.CAMERA,
  PERMISSIONS.IOS.MICROPHONE,
]);

const cameraOk = statuses[PERMISSIONS.IOS.CAMERA] === 'granted';
const micOk = statuses[PERMISSIONS.IOS.MICROPHONE] === 'granted';
```

### Open App Settings
```ts
import { openSettings } from 'react-native-permissions';

// Call when status is 'blocked' — only user can change from Settings
await openSettings();
```

### Complete Permission Request Flow

```ts
import { check, request, openSettings, PERMISSIONS, RESULTS } from 'react-native-permissions';
import { Platform, Alert } from 'react-native';

async function requestCameraPermission(): Promise<boolean> {
  const permission = Platform.select({
    ios: PERMISSIONS.IOS.CAMERA,
    android: PERMISSIONS.ANDROID.CAMERA,
  })!;

  const status = await check(permission);

  switch (status) {
    case RESULTS.UNAVAILABLE:
      Alert.alert('Camera not available on this device');
      return false;

    case RESULTS.GRANTED:
    case RESULTS.LIMITED:
      return true;

    case RESULTS.DENIED: {
      const result = await request(permission, {
        title: 'Camera Permission',
        message: 'We need camera access to take photos.',
        buttonPositive: 'OK',
        buttonNegative: 'Cancel',
      });
      return result === RESULTS.GRANTED;
    }

    case RESULTS.BLOCKED:
      Alert.alert(
        'Camera Permission Required',
        'Please enable camera access in Settings.',
        [
          { text: 'Cancel', style: 'cancel' },
          { text: 'Open Settings', onPress: openSettings },
        ]
      );
      return false;
  }
}
```

### Common Permission Constants

**iOS**
```ts
PERMISSIONS.IOS.CAMERA
PERMISSIONS.IOS.MICROPHONE
PERMISSIONS.IOS.PHOTO_LIBRARY
PERMISSIONS.IOS.PHOTO_LIBRARY_ADD_ONLY
PERMISSIONS.IOS.LOCATION_WHEN_IN_USE
PERMISSIONS.IOS.LOCATION_ALWAYS
PERMISSIONS.IOS.CONTACTS
PERMISSIONS.IOS.NOTIFICATIONS
PERMISSIONS.IOS.FACE_ID
```

**Android**
```ts
PERMISSIONS.ANDROID.CAMERA
PERMISSIONS.ANDROID.RECORD_AUDIO
PERMISSIONS.ANDROID.READ_MEDIA_IMAGES    // Android 13+
PERMISSIONS.ANDROID.READ_EXTERNAL_STORAGE // < Android 13
PERMISSIONS.ANDROID.ACCESS_FINE_LOCATION
PERMISSIONS.ANDROID.ACCESS_COARSE_LOCATION
PERMISSIONS.ANDROID.READ_CONTACTS
PERMISSIONS.ANDROID.POST_NOTIFICATIONS   // Android 13+
```

---

## Expo Permissions (Managed Workflow)

Each Expo module handles its own permissions — no separate library needed.

```ts
// Camera
import { useCameraPermissions } from 'expo-camera';
const [permission, requestPermission] = useCameraPermissions();

// Location
import { useForegroundPermissions } from 'expo-location';
const [status, requestPermission] = useForegroundPermissions();

// Notifications
import { requestPermissionsAsync, getPermissionsAsync } from 'expo-notifications';
const { status } = await requestPermissionsAsync();
```

---

## Best Practices

1. **Explain before requesting** — show a rationale screen/modal before triggering the OS prompt. You only get one native prompt; after denial you must direct users to Settings.

2. **Check before request** — always `check()` first to avoid showing the OS dialog if already granted.

3. **Handle `blocked` gracefully** — the user explicitly denied twice. Don't spam; show a clear path to Settings.

4. **Request just-in-time** — request permissions when the feature is used, not at app launch.

5. **iOS location** — always start with `LOCATION_WHEN_IN_USE`. Only request `LOCATION_ALWAYS` when truly required (background tracking), and Apple review requires strong justification.

6. **Android 13+ media** — use `READ_MEDIA_IMAGES`, `READ_MEDIA_VIDEO`, `READ_MEDIA_AUDIO` instead of `READ_EXTERNAL_STORAGE` on Android 13+.
