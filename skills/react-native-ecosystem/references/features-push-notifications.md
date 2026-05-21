# Features — Push Notifications

Push notifications require platform-specific setup: APNs (iOS) + FCM (Android).

## Option A — Firebase (React Native Firebase)

Best for bare React Native or complex notification needs (analytics, topics, data payloads).

### Installation
```bash
npm install @react-native-firebase/app @react-native-firebase/messaging
```

Add `google-services.json` (Android) and `GoogleService-Info.plist` (iOS).

### Request Permission (iOS)
```ts
import messaging from '@react-native-firebase/messaging';

async function requestNotificationPermission() {
  const authStatus = await messaging().requestPermission();
  return (
    authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
    authStatus === messaging.AuthorizationStatus.PROVISIONAL
  );
}
```

### Get FCM Token

```ts
async function registerForPushNotifications() {
  // iOS — must request permission first
  const granted = await requestNotificationPermission();
  if (!granted) return;

  // Get token
  const token = await messaging().getToken();
  await sendTokenToServer(token);

  // Refresh token listener
  return messaging().onTokenRefresh(async (newToken) => {
    await sendTokenToServer(newToken);
  });
}
```

### Foreground Messages

```ts
useEffect(() => {
  const unsubscribe = messaging().onMessage(async (remoteMessage) => {
    // App is in foreground — FCM does not auto-show UI; display it yourself
    showInAppNotification({
      title: remoteMessage.notification?.title,
      body: remoteMessage.notification?.body,
    });
  });
  return unsubscribe;
}, []);
```

### Background & Quit State

```ts
// Register outside of component tree (e.g., index.js)
// Runs when app is in background or closed
messaging().setBackgroundMessageHandler(async (remoteMessage) => {
  console.log('Background message:', remoteMessage);
  // Do minimal work here — no UI updates
});
```

### Notification Tap Handling

```ts
useEffect(() => {
  // App opened from background via notification tap
  const unsubscribe = messaging().onNotificationOpenedApp((remoteMessage) => {
    handleNotificationNavigation(remoteMessage.data);
  });

  // App opened from QUIT state via notification tap
  messaging()
    .getInitialNotification()
    .then((remoteMessage) => {
      if (remoteMessage) handleNotificationNavigation(remoteMessage.data);
    });

  return unsubscribe;
}, []);

function handleNotificationNavigation(data?: Record<string, string>) {
  if (data?.screen === 'chat') {
    navigation.navigate('Chat', { chatId: data.chatId });
  }
}
```

### Android Notification Channels (Android 8+)

```ts
import notifee from '@notifee/react-native'; // companion lib for display

async function createChannels() {
  await notifee.createChannel({
    id: 'messages',
    name: 'Messages',
    importance: AndroidImportance.HIGH,
    sound: 'default',
    vibration: true,
  });
}
```

### Subscribe to Topics

```ts
// Subscribe a device to a topic (no backend token management needed)
await messaging().subscribeToTopic('breaking-news');
await messaging().unsubscribeFromTopic('breaking-news');
```

---

## Option B — Expo Notifications (Expo Managed/Bare)

Simpler setup, works with Expo Go for testing.

```bash
npx expo install expo-notifications expo-device expo-constants
```

### Request Permission & Get Token

```ts
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import Constants from 'expo-constants';

async function registerForExpoPushNotifications(): Promise<string | null> {
  if (!Device.isDevice) {
    console.warn('Must use a physical device for push notifications');
    return null;
  }

  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') return null;

  const projectId = Constants.expoConfig?.extra?.eas?.projectId;
  const { data: token } = await Notifications.getExpoPushTokenAsync({ projectId });
  return token;
}
```

### Configure Notification Handling

```ts
// Set this before any listeners — controls foreground behavior
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
});
```

### Listeners

```ts
useEffect(() => {
  // Received while app is foregrounded
  const receivedSub = Notifications.addNotificationReceivedListener((notification) => {
    console.log('Received:', notification);
  });

  // User tapped notification
  const responseSub = Notifications.addNotificationResponseReceivedListener((response) => {
    const data = response.notification.request.content.data;
    handleNavigationFromNotification(data);
  });

  return () => {
    receivedSub.remove();
    responseSub.remove();
  };
}, []);
```

### Schedule Local Notification

```ts
await Notifications.scheduleNotificationAsync({
  content: {
    title: 'Reminder',
    body: 'Time to check your goals!',
    data: { screen: 'Goals' },
    sound: true,
  },
  trigger: {
    seconds: 60,       // fire in 60 seconds
    repeats: false,
  },
});

// Cancel all scheduled
await Notifications.cancelAllScheduledNotificationsAsync();
```

---

## Deep Link on Notification Tap

```ts
// When notification carries a URL or route
messaging().onNotificationOpenedApp((remoteMessage) => {
  const url = remoteMessage.data?.url;
  if (url) Linking.openURL(url);
});
```

---

## iOS Capabilities Checklist

- Enable **Push Notifications** capability in Xcode
- Enable **Background Modes → Remote notifications** for background delivery
- APNs key or certificate uploaded to Firebase Console / Expo EAS
