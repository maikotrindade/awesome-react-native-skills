---
name: react-native-accessibility
description: Accessibility APIs for making apps accessible to assistive technologies. Use when implementing screen reader support, accessibility labels, or ensuring app accessibility.
---

# Accessibility

React Native provides APIs to make apps accessible to assistive technologies like VoiceOver (iOS) and TalkBack (Android).

## Basic Accessibility Properties

### accessible

Marks a view as discoverable by assistive technologies.

```tsx
<View accessible={true}>
  <Text>This view is accessible</Text>
</View>
```

**Default**: All touchable elements are accessible.

### accessibilityLabel

Provides a label read by screen readers.

```tsx
<TouchableOpacity
  accessible={true}
  accessibilityLabel="Tap to go back"
  onPress={handlePress}
>
  <Text>Back</Text>
</TouchableOpacity>
```

**Note**: If no `accessibilityLabel` is provided, screen readers concatenate all Text children.

### accessibilityHint

Provides additional context about what happens when the element is activated.

```tsx
<TouchableOpacity
  accessible={true}
  accessibilityLabel="Delete"
  accessibilityHint="Deletes the current item"
  onPress={handleDelete}
>
  <Text>Delete</Text>
</TouchableOpacity>
```

### accessibilityRole

Describes the purpose of the element.

```tsx
<View accessibilityRole="button">
  <Text>Click me</Text>
</View>

<View accessibilityRole="header">
  <Text>Title</Text>
</View>
```

**Common roles:**
- `button`, `link`, `header`, `text`, `image`, `none`
- `search`, `keyboardkey`, `adjustable`, `summary`
- `alert`, `checkbox`, `combobox`, `menu`, `menubar`
- `menuitem`, `progressbar`, `radio`, `radiogroup`
- `scrollbar`, `spinbutton`, `switch`, `tab`, `tablist`
- `timer`, `toolbar`

### accessibilityState

Describes the current state of the element.

```tsx
<View
  accessibilityRole="checkbox"
  accessibilityState={{ checked: true, disabled: false }}
>
  <Text>Option</Text>
</View>
```

**State properties:**
- `selected`, `disabled`, `checked`, `busy`, `expanded`

### accessibilityValue

Provides current value for adjustable elements.

```tsx
<Slider
  accessibilityRole="adjustable"
  accessibilityValue={{ min: 0, max: 100, now: 50, text: '50%' }}
/>
```

## Form Accessibility

### accessibilityLabelledBy (Android)

Links input to its label.

```tsx
<View>
  <Text nativeID="emailLabel">Email Address</Text>
  <TextInput
    accessibilityLabel="Email input"
    accessibilityLabelledBy="emailLabel"
  />
</View>
```

### accessibilityLiveRegion (Android)

Announces dynamic content changes.

```tsx
<View accessibilityLiveRegion="polite">
  <Text>{dynamicContent}</Text>
</View>
```

**Values:**
- `none` - No announcements
- `polite` - Announce when user is idle
- `assertive` - Interrupt to announce

## Image Accessibility

### accessibilityLabel for Images

```tsx
<Image
  source={require('./logo.png')}
  accessibilityLabel="Company logo"
  accessible={true}
/>
```

### accessibilityIgnoresInvertColors (iOS)

Prevent color inversion for images.

```tsx
<Image
  source={require('./photo.png')}
  accessibilityIgnoresInvertColors={true}
/>
```

## Touchable Accessibility

Touchable components are accessible by default.

```tsx
<TouchableOpacity
  accessible={true}
  accessibilityRole="button"
  accessibilityLabel="Submit form"
  accessibilityHint="Submits the form data"
  onPress={handleSubmit}
>
  <Text>Submit</Text>
</TouchableOpacity>
```

## Text Accessibility

### allowFontScaling

Control whether text scales with system font size.

```tsx
<Text allowFontScaling={false}>
  Fixed size text
</Text>
```

### maxFontSizeMultiplier

Limit maximum font scale.

```tsx
<Text maxFontSizeMultiplier={1.5}>
  Text that scales up to 1.5x
</Text>
```

## AccessibilityInfo

Query accessibility features and state.

```tsx
import { AccessibilityInfo } from 'react-native';

// Check if screen reader is enabled
const isScreenReaderEnabled = await AccessibilityInfo.isScreenReaderEnabled();

// Listen to changes
useEffect(() => {
  const subscription = AccessibilityInfo.addEventListener(
    'change',
    (isEnabled) => {
      console.log('Screen reader:', isEnabled);
    }
  );

  return () => subscription.remove();
}, []);

// Check reduce motion preference
const isReduceMotionEnabled = await AccessibilityInfo.isReduceMotionEnabled();

// Check reduce transparency preference (iOS)
const isReduceTransparencyEnabled = 
  await AccessibilityInfo.isReduceTransparencyEnabled();
```

## Best Practices

1. **Always provide accessibilityLabel** - For interactive elements
2. **Use semantic roles** - `button`, `header`, `link`, etc.
3. **Describe state** - Use `accessibilityState` for checkboxes, switches
4. **Provide hints** - When action result isn't clear from label
5. **Test with screen readers** - VoiceOver (iOS) and TalkBack (Android)
6. **Allow font scaling** - Don't disable unless necessary
7. **Use proper hierarchy** - Headers, sections, landmarks
8. **Handle dynamic content** - Use `accessibilityLiveRegion` for updates
9. **Test color contrast** - Ensure text is readable
10. **Avoid redundant labels** - Don't repeat visible text

## Common Patterns

### Accessible Button

```tsx
const AccessibleButton = ({ label, hint, onPress, disabled }) => (
  <TouchableOpacity
    accessible={true}
    accessibilityRole="button"
    accessibilityLabel={label}
    accessibilityHint={hint}
    accessibilityState={{ disabled }}
    onPress={onPress}
    disabled={disabled}
  >
    <Text>{label}</Text>
  </TouchableOpacity>
);
```

### Accessible Form

```tsx
<View>
  <Text nativeID="emailLabel" accessibilityRole="text">
    Email Address
  </Text>
  <TextInput
    accessibilityLabel="Email"
    accessibilityLabelledBy="emailLabel"
    accessibilityHint="Enter your email address"
  />
</View>
```

### Dynamic Content Announcement

```tsx
const [message, setMessage] = useState('');

<View accessibilityLiveRegion="polite">
  <Text>{message}</Text>
</View>

// When message changes, screen reader announces it
setMessage('Form submitted successfully');
```

<!--
Source references:
- https://reactnative.dev/docs/accessibility
- https://reactnative.dev/docs/accessibilityinfo
-->
