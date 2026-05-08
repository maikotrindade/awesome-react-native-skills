---
name: react-native-keyboard
description: Keyboard API for handling keyboard events and dismissing keyboard. Use when building forms, handling keyboard visibility, or managing keyboard interactions.
---

# Keyboard API

The `Keyboard` module provides APIs to control keyboard behavior and listen to keyboard events.

## Dismissing Keyboard

```tsx
import { Keyboard } from 'react-native';

// Dismiss keyboard programmatically
Keyboard.dismiss();
```

### Common Usage

```tsx
import { Keyboard, TextInput, TouchableWithoutFeedback, View } from 'react-native';

const App = () => (
  <TouchableWithoutFeedback onPress={Keyboard.dismiss}>
    <View>
      <TextInput placeholder="Tap outside to dismiss" />
    </View>
  </TouchableWithoutFeedback>
);
```

## Keyboard Events

Listen to keyboard show/hide events.

```tsx
import { useEffect } from 'react';
import { Keyboard } from 'react-native';

const App = () => {
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

  return <View />;
};
```

### Event Types

```tsx
// iOS only - fires before keyboard appears
'keyboardWillShow'

// Fires after keyboard appears
'keyboardDidShow'

// iOS only - fires before keyboard hides
'keyboardWillHide'

// Fires after keyboard hides
'keyboardDidHide'
```

### Event Data

```tsx
Keyboard.addListener('keyboardDidShow', (event) => {
  const {
    endCoordinates: {
      height,      // Keyboard height
      width,       // Keyboard width
      screenX,     // X position
      screenY,     // Y position
    },
    duration,      // Animation duration (iOS)
    easing,        // Animation easing (iOS)
  } = event;
});
```

## KeyboardAvoidingView

Automatically adjusts view when keyboard appears.

```tsx
import { KeyboardAvoidingView, Platform } from 'react-native';

<KeyboardAvoidingView
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
  style={{ flex: 1 }}
>
  <TextInput />
</KeyboardAvoidingView>
```

### Behavior Options

```tsx
<KeyboardAvoidingView
  behavior="padding"   // 'padding' | 'height' | 'position'
  keyboardVerticalOffset={64}  // Additional offset
/>
```

**Behavior values:**
- `padding` - Add padding when keyboard appears (iOS)
- `height` - Resize view height (Android)
- `position` - Move view position

## Common Patterns

### Dismiss on Submit

```tsx
<TextInput
  onSubmitEditing={Keyboard.dismiss}
  returnKeyType="done"
/>
```

### Dismiss on Outside Tap

```tsx
<TouchableWithoutFeedback onPress={Keyboard.dismiss}>
  <View style={{ flex: 1 }}>
    <TextInput />
  </View>
</TouchableWithoutFeedback>
```

### Adjust Layout on Keyboard

```tsx
const [keyboardHeight, setKeyboardHeight] = useState(0);

useEffect(() => {
  const showSubscription = Keyboard.addListener('keyboardDidShow', (e) => {
    setKeyboardHeight(e.endCoordinates.height);
  });

  const hideSubscription = Keyboard.addListener('keyboardDidHide', () => {
    setKeyboardHeight(0);
  });

  return () => {
    showSubscription.remove();
    hideSubscription.remove();
  };
}, []);

<View style={{ paddingBottom: keyboardHeight }}>
  <TextInput />
</View>
```

### Form with Keyboard Avoidance

```tsx
<KeyboardAvoidingView
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
  style={{ flex: 1 }}
>
  <ScrollView contentContainerStyle={{ flexGrow: 1 }}>
    <TextInput placeholder="Email" />
    <TextInput placeholder="Password" secureTextEntry />
    <Button title="Submit" onPress={handleSubmit} />
  </ScrollView>
</KeyboardAvoidingView>
```

## Best Practices

1. **Use KeyboardAvoidingView** - For forms and input screens
2. **Dismiss on submit** - Set `onSubmitEditing={Keyboard.dismiss}`
3. **Handle platform differences** - iOS uses 'padding', Android uses 'height'
4. **Listen to events** - Adjust layout based on keyboard visibility
5. **Test on real devices** - Keyboard behavior differs from simulator
6. **Use ScrollView** - Combine with KeyboardAvoidingView for long forms
7. **Set keyboardVerticalOffset** - Adjust for navigation headers

## Common Patterns

### Keyboard Hook

```tsx
const useKeyboard = () => {
  const [keyboardVisible, setKeyboardVisible] = useState(false);
  const [keyboardHeight, setKeyboardHeight] = useState(0);

  useEffect(() => {
    const showSubscription = Keyboard.addListener('keyboardDidShow', (e) => {
      setKeyboardVisible(true);
      setKeyboardHeight(e.endCoordinates.height);
    });

    const hideSubscription = Keyboard.addListener('keyboardDidHide', () => {
      setKeyboardVisible(false);
      setKeyboardHeight(0);
    });

    return () => {
      showSubscription.remove();
      hideSubscription.remove();
    };
  }, []);

  return { keyboardVisible, keyboardHeight };
};
```

### Dismissible Form

```tsx
const Form = () => {
  return (
    <TouchableWithoutFeedback onPress={Keyboard.dismiss}>
      <KeyboardAvoidingView
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        style={{ flex: 1 }}
      >
        <ScrollView>
          {/* Form fields */}
        </ScrollView>
      </KeyboardAvoidingView>
    </TouchableWithoutFeedback>
  );
};
```

<!--
Source references:
- https://reactnative.dev/docs/keyboard
- https://reactnative.dev/docs/keyboardavoidingview
-->
