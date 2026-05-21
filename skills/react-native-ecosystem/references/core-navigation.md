# Core Navigation — React Navigation v7

React Navigation v7 is the standard routing solution for React Native. Always prefer `createNativeStackNavigator` (uses native platform primitives) over the JS-based `createStackNavigator`.

## Installation

```bash
npm install @react-navigation/native @react-navigation/native-stack
npm install react-native-screens react-native-safe-area-context
# For tabs
npm install @react-navigation/bottom-tabs
# For drawer
npm install @react-navigation/drawer react-native-gesture-handler react-native-reanimated
```

## Basic Setup

```tsx
// App.tsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

export type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Settings: { section?: string };
};

const Stack = createNativeStackNavigator<RootStackParamList>();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
        <Stack.Screen
          name="Settings"
          component={SettingsScreen}
          options={{ title: 'Settings', headerShown: true }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

## Typed Navigation Hooks

```tsx
import { useNavigation, useRoute } from '@react-navigation/native';
import type { NativeStackNavigationProp, NativeStackScreenProps } from '@react-navigation/native-stack';

// In a screen component
type Props = NativeStackScreenProps<RootStackParamList, 'Profile'>;

export function ProfileScreen({ navigation, route }: Props) {
  const { userId } = route.params;
  return <Button onPress={() => navigation.navigate('Settings')} title="Go" />;
}

// In a non-screen component (useNavigation)
type NavProp = NativeStackNavigationProp<RootStackParamList>;

function SomeButton() {
  const navigation = useNavigation<NavProp>();
  return <Button onPress={() => navigation.navigate('Home')} title="Home" />;
}
```

## Bottom Tabs

```tsx
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

export type TabParamList = {
  Feed: undefined;
  Search: undefined;
  Profile: undefined;
};

const Tab = createBottomTabNavigator<TabParamList>();

function TabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ color, size }) => {
          const icons: Record<keyof TabParamList, string> = {
            Feed: 'home',
            Search: 'search',
            Profile: 'person',
          };
          return <Icon name={icons[route.name]} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: 'gray',
      })}
    >
      <Tab.Screen name="Feed" component={FeedScreen} />
      <Tab.Screen name="Search" component={SearchScreen} />
      <Tab.Screen
        name="Profile"
        component={ProfileScreen}
        options={{ tabBarBadge: 3 }}
      />
    </Tab.Navigator>
  );
}
```

## Drawer Navigator

```tsx
import { createDrawerNavigator } from '@react-navigation/drawer';

const Drawer = createDrawerNavigator();

function DrawerNavigator() {
  return (
    <Drawer.Navigator
      screenOptions={{
        drawerType: 'front',       // 'front' | 'back' | 'slide' | 'permanent'
        drawerPosition: 'left',
        swipeEnabled: true,
      }}
    >
      <Drawer.Screen name="Home" component={HomeScreen} />
      <Drawer.Screen name="Settings" component={SettingsScreen} />
    </Drawer.Navigator>
  );
}
```

## Nested Navigators

```tsx
// Tab inside a Stack — the root Stack wraps a Tab navigator
export type RootStackParamList = {
  Auth: undefined;
  Main: NavigatorScreenParams<TabParamList>; // typed nesting
  Modal: { content: string };
};

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Auth" component={AuthScreen} options={{ headerShown: false }} />
        <Stack.Screen name="Main" component={TabNavigator} options={{ headerShown: false }} />
        <Stack.Screen name="Modal" component={ModalScreen} options={{ presentation: 'modal' }} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

// Navigate into nested screen
navigation.navigate('Main', { screen: 'Search' });
```

## Composite Navigation Type (nested navigators)

```tsx
import type { CompositeNavigationProp } from '@react-navigation/native';

type HomeNavProp = CompositeNavigationProp<
  BottomTabNavigationProp<TabParamList, 'Feed'>,
  NativeStackNavigationProp<RootStackParamList>
>;
```

## Common Screen Options

```tsx
<Stack.Screen
  name="Detail"
  component={DetailScreen}
  options={{
    title: 'Detail',
    headerShown: true,
    headerBackTitle: 'Back',       // iOS only
    presentation: 'modal',         // 'card' | 'modal' | 'transparentModal' | 'fullScreenModal'
    gestureEnabled: true,
    animation: 'slide_from_right', // 'default' | 'fade' | 'flip' | 'none' | 'slide_from_right'
    headerRight: () => <Button title="Done" onPress={handleDone} />,
  }}
/>
```

## Key Hooks

| Hook | Purpose |
|------|---------|
| `useNavigation()` | Access navigation object in non-screen components |
| `useRoute()` | Access route params in non-screen components |
| `useFocusEffect(cb)` | Run effect when screen is focused (re-runs on re-focus) |
| `useIsFocused()` | Boolean — whether the screen is currently focused |
| `useNavigationState(selector)` | Subscribe to navigation state |

```tsx
import { useFocusEffect } from '@react-navigation/native';
import { useCallback } from 'react';

function ProfileScreen() {
  useFocusEffect(
    useCallback(() => {
      fetchProfile(); // runs every time screen comes into focus
      return () => cancelFetch(); // cleanup on blur
    }, [])
  );
}
```

## Passing and Updating Params

```tsx
// Navigate with params
navigation.navigate('Profile', { userId: '42' });

// Update params in-screen (does not trigger re-navigation)
navigation.setParams({ userId: 'updated' });

// Go back
navigation.goBack();
navigation.popToTop(); // NativeStack only — goes to first screen in stack
```

## Header Customization

```tsx
// Dynamic header options from within a screen
useLayoutEffect(() => {
  navigation.setOptions({
    title: user.name,
    headerRight: () => (
      <Pressable onPress={handleEdit}>
        <Text>Edit</Text>
      </Pressable>
    ),
  });
}, [navigation, user.name]);
```
