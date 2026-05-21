# Best Practices — Accessibility

React Native maps `accessibility*` props to native a11y APIs: UIAccessibility (iOS) and AccessibilityNodeInfo (Android).

---

## Core Props

### accessibilityRole

Tells screen readers what type of element this is:

```tsx
// Common roles
<Pressable accessibilityRole="button">
<TextInput accessibilityRole="none" />       // override default
<View accessibilityRole="header" />
<TouchableOpacity accessibilityRole="link">
<Switch accessibilityRole="switch" />
<View accessibilityRole="progressbar" />
<Image accessibilityRole="image" />
```

Full list: `none`, `button`, `link`, `search`, `image`, `keyboardkey`, `text`, `adjustable`, `imagebutton`, `header`, `summary`, `alert`, `checkbox`, `combobox`, `menu`, `menubar`, `menuitem`, `progressbar`, `radio`, `radiogroup`, `scrollbar`, `spinbutton`, `switch`, `tab`, `tablist`, `timer`, `toolbar`

### accessibilityLabel + accessibilityHint

```tsx
<Pressable
  accessibilityLabel="Delete post"           // replaces visual text for screen reader
  accessibilityHint="Removes the post permanently"  // extra context, announced after label
  onPress={handleDelete}
>
  <Icon name="trash" />
</Pressable>

// For images
<Image
  source={user.avatar}
  accessibilityLabel={`${user.name}'s profile picture`}
  accessible={true}
/>
```

### accessibilityState

```tsx
<Pressable
  accessibilityRole="button"
  accessibilityState={{
    disabled: isLoading,
    selected: isSelected,
    checked: isChecked,     // for checkboxes/radios
    busy: isLoading,        // announces "busy" to screen reader
    expanded: isExpanded,   // for accordions/dropdowns
  }}
>
```

### accessibilityValue

```tsx
// For sliders, progress bars, steppers
<Slider
  accessibilityRole="adjustable"
  accessibilityValue={{ min: 0, max: 100, now: value, text: `${value}%` }}
/>
```

---

## Grouping & Focus Control

### accessible

When `accessible={true}`, the component and all its children are treated as a single focusable element:

```tsx
<View accessible={true} accessibilityLabel="User card: Maiko, followed by 3 posts">
  <Image source={avatar} />
  <Text>Maiko</Text>
  <Text>3 posts</Text>
</View>
```

### importantForAccessibility (Android)

```tsx
// Hide decorative elements from accessibility tree
<Image
  source={decorativeBackground}
  importantForAccessibility="no"
/>

// Hide entire subtree
<View importantForAccessibility="no-hide-descendants">
  {/* decorative animation */}
</View>
```

### accessibilityViewIsModal (iOS)

For modals/overlays — prevents VoiceOver from reading content behind the modal:

```tsx
<View style={styles.modal} accessibilityViewIsModal={true}>
  {/* modal content */}
</View>
```

---

## Dynamic Content

### accessibilityLiveRegion

Announces changes to dynamic content without focus change:

```tsx
// 'polite' — announce when user is idle
// 'assertive' — interrupt and announce immediately
<Text accessibilityLiveRegion="polite">
  {statusMessage}
</Text>

// Common use: countdown timers, status updates, validation errors
<Text accessibilityLiveRegion="assertive" style={styles.error}>
  {errorMessage}
</Text>
```

---

## Focus Management

### setAccessibilityFocus

Programmatically move focus to a specific element (e.g., when a modal opens):

```tsx
import { AccessibilityInfo, findNodeHandle } from 'react-native';

const headingRef = useRef<Text>(null);

useEffect(() => {
  if (modalVisible) {
    const reactTag = findNodeHandle(headingRef.current);
    if (reactTag) AccessibilityInfo.setAccessibilityFocus(reactTag);
  }
}, [modalVisible]);

return <Text ref={headingRef}>Modal Title</Text>;
```

### announceForAccessibility

Announce a message without moving focus:

```tsx
AccessibilityInfo.announceForAccessibility('Item added to cart');
```

---

## Screen Reader Detection

```ts
import { AccessibilityInfo } from 'react-native';

// Check once
const isEnabled = await AccessibilityInfo.isScreenReaderEnabled();

// Subscribe to changes
const subscription = AccessibilityInfo.addEventListener(
  'screenReaderChanged',
  (isEnabled) => setScreenReaderEnabled(isEnabled)
);
return () => subscription.remove();
```

---

## Touch Target Sizes

Minimum sizes per platform guidelines:
- **iOS**: 44×44 pt
- **Android**: 48×48 dp

```tsx
// Expand touch area without changing visual size
<Pressable
  style={styles.iconButton}
  hitSlop={{ top: 12, bottom: 12, left: 12, right: 12 }}
>
  <Icon name="close" size={20} />
</Pressable>

// Or use minHeight/minWidth with alignment
const styles = StyleSheet.create({
  iconButton: {
    minWidth: 44,
    minHeight: 44,
    alignItems: 'center',
    justifyContent: 'center',
  },
});
```

---

## Reduce Motion

Respect the user's "Reduce Motion" system setting:

```tsx
import { AccessibilityInfo } from 'react-native';
import { useReducedMotion } from 'react-native-reanimated';

function AnimatedCard() {
  const reduceMotion = useReducedMotion(); // Reanimated built-in hook

  const entering = reduceMotion ? FadeIn : SlideInRight;
  return <Animated.View entering={entering}>{/* ... */}</Animated.View>;
}
```

---

## Testing Checklist

**Manual (device)**
- [ ] VoiceOver (iOS): Swipe through all interactive elements, verify labels and roles are meaningful
- [ ] TalkBack (Android): Same check on Android
- [ ] Dynamic text size: test at 2× font size — verify no overflow or clipping
- [ ] Minimum touch targets: all tappable elements meet 44×44pt / 48×48dp
- [ ] Color contrast: minimum 4.5:1 for normal text, 3:1 for large text

**Automated**
- [ ] `accessibilityRole` present on all interactive elements
- [ ] No interactive element has only an icon without a label
- [ ] `accessibilityLabel` covers icon-only buttons
