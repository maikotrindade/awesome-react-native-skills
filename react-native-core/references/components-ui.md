---
name: react-native-ui-components
description: Additional UI components - Modal, Switch, ActivityIndicator, RefreshControl. Use when building dialogs, loading states, pull-to-refresh, or toggle switches.
---

# UI Components

Additional React Native components for common UI patterns.

## Modal

Presents content above an enclosing view. Useful for dialogs, alerts, and overlays.

```tsx
import { useState } from 'react';
import { Modal, View, Text, Pressable, StyleSheet } from 'react-native';

const App = () => {
  const [modalVisible, setModalVisible] = useState(false);

  return (
    <>
      <Pressable onPress={() => setModalVisible(true)}>
        <Text>Show Modal</Text>
      </Pressable>

      <Modal
        animationType="slide"
        transparent={true}
        visible={modalVisible}
        onRequestClose={() => setModalVisible(false)}
      >
        <View style={styles.centeredView}>
          <View style={styles.modalView}>
            <Text>Modal Content</Text>
            <Pressable onPress={() => setModalVisible(false)}>
              <Text>Close</Text>
            </Pressable>
          </View>
        </View>
      </Modal>
    </>
  );
};
```

### Modal Props

```tsx
<Modal
  visible={true}                    // Control visibility
  animationType="slide"            // 'none' | 'slide' | 'fade'
  transparent={false}              // Transparent background
  onRequestClose={() => {}}        // Android back button handler
  onShow={() => {}}                // Called when modal is shown
  onDismiss={() => {}}             // Called when modal is dismissed
  presentationStyle="pageSheet"    // iOS: 'fullScreen' | 'pageSheet' | 'formSheet' | 'overFullScreen'
  hardwareAccelerated={true}       // Android hardware acceleration
  statusBarTranslucent={true}      // Android status bar
/>
```

### Common Patterns

**Centered Modal:**

```tsx
<Modal transparent visible={visible}>
  <View style={styles.centeredView}>
    <View style={styles.modalView}>
      {/* Content */}
    </View>
  </View>
</Modal>

const styles = StyleSheet.create({
  centeredView: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: 'rgba(0,0,0,0.5)',
  },
  modalView: {
    backgroundColor: 'white',
    borderRadius: 20,
    padding: 35,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 4,
    elevation: 5,
  },
});
```

**Bottom Sheet:**

```tsx
<Modal
  visible={visible}
  transparent
  animationType="slide"
  onRequestClose={onClose}
>
  <Pressable style={styles.backdrop} onPress={onClose}>
    <View style={styles.bottomSheet}>
      {/* Content */}
    </View>
  </Pressable>
</Modal>
```

## Switch

Renders a boolean toggle input. Controlled component requiring `value` and `onValueChange`.

```tsx
import { useState } from 'react';
import { Switch, View } from 'react-native';

const App = () => {
  const [isEnabled, setIsEnabled] = useState(false);

  return (
    <Switch
      value={isEnabled}
      onValueChange={setIsEnabled}
      trackColor={{ false: '#767577', true: '#81b0ff' }}
      thumbColor={isEnabled ? '#f5dd4b' : '#f4f3f4'}
      ios_backgroundColor="#3e3e3e"
    />
  );
};
```

### Switch Props

```tsx
<Switch
  value={isEnabled}                 // Controlled value
  onValueChange={setIsEnabled}     // Change handler
  disabled={false}                  // Disable toggle
  trackColor={{                    // Track colors
    false: '#767577',              // Unchecked color
    true: '#81b0ff'                // Checked color
  }}
  thumbColor="#f5dd4b"             // Thumb color (Android)
  ios_backgroundColor="#3e3e3e"    // iOS background color
/>
```

## ActivityIndicator

Displays a circular loading indicator.

```tsx
import { ActivityIndicator, View } from 'react-native';

const App = () => (
  <View>
    <ActivityIndicator />
    <ActivityIndicator size="large" />
    <ActivityIndicator size="small" color="#0000ff" />
  </View>
);
```

### ActivityIndicator Props

```tsx
<ActivityIndicator
  animating={true}                  // Show/hide indicator
  size="large"                      // 'small' | 'large'
  color="#0000ff"                   // Color (default: system accent)
  hidesWhenStopped={true}           // iOS: hide when not animating
/>
```

### Common Patterns

**Loading State:**

```tsx
{isLoading ? (
  <ActivityIndicator size="large" />
) : (
  <Content />
)}
```

**Full Screen Loading:**

```tsx
{loading && (
  <View style={styles.loadingContainer}>
    <ActivityIndicator size="large" />
    <Text>Loading...</Text>
  </View>
)}
```

## RefreshControl

Adds pull-to-refresh functionality to ScrollView or FlatList.

```tsx
import { useState, useCallback } from 'react';
import { ScrollView, RefreshControl } from 'react-native';

const App = () => {
  const [refreshing, setRefreshing] = useState(false);

  const onRefresh = useCallback(async () => {
    setRefreshing(true);
    await fetchData();
    setRefreshing(false);
  }, []);

  return (
    <ScrollView
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
      }
    >
      {/* Content */}
    </ScrollView>
  );
};
```

### RefreshControl Props

```tsx
<RefreshControl
  refreshing={refreshing}           // Controlled refreshing state
  onRefresh={() => {}}              // Refresh handler
  colors={['#ff0000']}              // Android: refresh indicator colors
  tintColor="#ff0000"               // iOS: refresh indicator color
  title="Pull to refresh"           // iOS: title text
  titleColor="#000000"              // iOS: title color
  progressBackgroundColor="#ffffff" // Android: background color
/>
```

### With FlatList

```tsx
<FlatList
  data={data}
  renderItem={renderItem}
  refreshControl={
    <RefreshControl
      refreshing={refreshing}
      onRefresh={onRefresh}
    />
  }
/>
```

## Best Practices

1. **Modal**: Always handle `onRequestClose` for Android back button
2. **Switch**: Use controlled component pattern with `value` and `onValueChange`
3. **ActivityIndicator**: Show during async operations, hide when complete
4. **RefreshControl**: Set `refreshing` to `true` immediately in `onRefresh`
5. **Test on both platforms** - Modal and Switch behavior differs
6. **Use transparent Modal** for overlays and dialogs
7. **Provide visual feedback** - Loading states improve UX

## Common Patterns

### Confirmation Modal

```tsx
const [showConfirm, setShowConfirm] = useState(false);

<Modal visible={showConfirm} transparent>
  <View style={styles.centeredView}>
    <View style={styles.modalView}>
      <Text>Are you sure?</Text>
      <View style={styles.buttons}>
        <Button title="Cancel" onPress={() => setShowConfirm(false)} />
        <Button title="Confirm" onPress={handleConfirm} />
      </View>
    </View>
  </View>
</Modal>
```

### Settings Toggle

```tsx
const [notifications, setNotifications] = useState(false);

<View style={styles.settingRow}>
  <Text>Enable Notifications</Text>
  <Switch
    value={notifications}
    onValueChange={setNotifications}
  />
</View>
```

### Pull to Refresh List

```tsx
const [refreshing, setRefreshing] = useState(false);

const handleRefresh = async () => {
  setRefreshing(true);
  try {
    await refetchData();
  } finally {
    setRefreshing(false);
  }
};

<FlatList
  data={items}
  refreshControl={
    <RefreshControl refreshing={refreshing} onRefresh={handleRefresh} />
  }
/>
```

<!--
Source references:
- https://reactnative.dev/docs/modal
- https://reactnative.dev/docs/switch
- https://reactnative.dev/docs/activityindicator
- https://reactnative.dev/docs/refreshcontrol
-->
