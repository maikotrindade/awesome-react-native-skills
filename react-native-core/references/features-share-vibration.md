---
name: react-native-share-vibration
description: Share API for native sharing and Vibration API for haptic feedback. Use when implementing share functionality or providing vibration feedback.
---

# Share and Vibration

React Native provides APIs for native sharing and device vibration.

## Share API

Open native share dialog to share content.

### Basic Usage

```tsx
import { Share, Alert } from 'react-native';

const shareContent = async () => {
  try {
    const result = await Share.share({
      message: 'React Native is awesome!',
    });

    if (result.action === Share.sharedAction) {
      if (result.activityType) {
        // Shared with specific activity type
        console.log('Shared via:', result.activityType);
      } else {
        // Shared
        console.log('Content shared');
      }
    } else if (result.action === Share.dismissedAction) {
      // Dismissed
      console.log('Share dismissed');
    }
  } catch (error) {
    Alert.alert('Error', error.message);
  }
};
```

### Share Options

```tsx
// iOS - title and url
await Share.share({
  message: 'Check out this link',
  url: 'https://example.com',
  title: 'Share Title',
});

// Android - message only
await Share.share({
  message: 'Check out this link: https://example.com',
});
```

### Share with Dialog Options (Android)

```tsx
await Share.share(
  {
    message: 'Share this content',
  },
  {
    dialogTitle: 'Share via',
    subject: 'Subject line', // Email subject
  }
);
```

### Share Result

```tsx
const result = await Share.share({ message: 'Hello' });

// result.action values:
// - Share.sharedAction - User shared
// - Share.dismissedAction - User dismissed

// result.activityType (iOS only):
// - Specific app/service used to share
```

## Vibration API

Provide haptic feedback through device vibration.

### Basic Vibration

```tsx
import { Vibration } from 'react-native';

// Vibrate once (default duration)
Vibration.vibrate();

// Vibrate for specific duration (Android only)
Vibration.vibrate(1000); // 1 second
```

### Vibration Pattern

```tsx
// Pattern: wait, vibrate, wait, vibrate, ...
const pattern = [0, 1000, 500, 1000];

Vibration.vibrate(pattern);
// Wait 0ms, vibrate 1000ms, wait 500ms, vibrate 1000ms
```

### Repeat Pattern

```tsx
// Repeat pattern until cancelled
Vibration.vibrate([0, 1000, 500, 1000], true);

// Cancel repeating vibration
Vibration.cancel();
```

### Platform Differences

**iOS:**
- Pattern alternates between wait and vibrate
- `[0, 1000, 500, 1000]` = wait 0ms, vibrate, wait 500ms, vibrate

**Android:**
- First value is initial delay
- `[0, 1000, 500, 1000]` = wait 0ms, vibrate 1000ms, wait 500ms, vibrate 1000ms

## Common Patterns

### Share Button

```tsx
const ShareButton = ({ content, title }) => {
  const handleShare = async () => {
    try {
      await Share.share({
        message: content,
        ...(title && { title }),
      });
    } catch (error) {
      console.error('Share failed:', error);
    }
  };

  return (
    <TouchableOpacity onPress={handleShare}>
      <Text>Share</Text>
    </TouchableOpacity>
  );
};
```

### Share Link

```tsx
const shareLink = async (url, title) => {
  if (Platform.OS === 'ios') {
    await Share.share({
      message: title,
      url: url,
      title: title,
    });
  } else {
    await Share.share({
      message: `${title}\n${url}`,
    });
  }
};
```

### Haptic Feedback

```tsx
const handlePress = () => {
  // Light vibration for feedback
  Vibration.vibrate(50);
  
  // Perform action
  doSomething();
};
```

### Success Vibration Pattern

```tsx
const successVibration = () => {
  Vibration.vibrate([0, 100, 50, 100]);
};
```

### Error Vibration Pattern

```tsx
const errorVibration = () => {
  Vibration.vibrate([0, 200, 100, 200, 100, 200]);
};
```

## Best Practices

1. **Handle share errors** - Wrap in try-catch
2. **Check share result** - Handle dismissed action
3. **Use appropriate vibration** - Don't overuse
4. **Test on device** - Vibration doesn't work in simulator
5. **Consider user preferences** - Some users disable vibration
6. **Platform differences** - Share API differs between iOS and Android
7. **Cancel long vibrations** - Cancel repeating patterns when done

## Common Patterns

### Share with Fallback

```tsx
const shareWithFallback = async (content) => {
  try {
    const result = await Share.share({ message: content });
    if (result.action === Share.dismissedAction) {
      // User cancelled - no action needed
    }
  } catch (error) {
    // Fallback: copy to clipboard
    Clipboard.setString(content);
    Alert.alert('Copied to clipboard');
  }
};
```

### Vibration Feedback Hook

```tsx
const useVibration = () => {
  const vibrate = useCallback((pattern) => {
    if (Platform.OS !== 'web') {
      Vibration.vibrate(pattern);
    }
  }, []);

  return {
    vibrate,
    success: () => vibrate([0, 100, 50, 100]),
    error: () => vibrate([0, 200, 100, 200]),
    light: () => vibrate(50),
  };
};
```

<!--
Source references:
- https://reactnative.dev/docs/share
- https://reactnative.dev/docs/vibration
-->
