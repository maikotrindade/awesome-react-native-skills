---
name: react-native-expo-best-practices-ui
description: Building native Expo UI — code style, expo-image, responsiveness, NativeTabs (SDK 55+), and Reanimated v3/v4.
---

# Building Native UI with Expo

## Code Style

- **Kebab-case filenames:** `user-profile.tsx`, not `UserProfile.tsx`
- **Routes in `app/` only:** Never co-locate components, types, utilities, or hooks in the `app/` directory — use `components/`, `lib/`, `hooks/` at the root level
- **Path aliases:** Configure `@/` in `tsconfig.json` so imports don't break on directory changes
- **`process.env.EXPO_OS`** — use instead of `Platform.OS` in shared code for better tree-shaking

```ts
// tsconfig.json
{
  "compilerOptions": {
    "paths": { "@/*": ["./*"] }
  }
}
```

## Library Preferences

| Use case | Preferred | Avoid |
|----------|-----------|-------|
| Images | `expo-image` | RN `Image` |
| Audio | `expo-audio` | `expo-av` (deprecated SDK 55) |
| Video | `expo-video` | `expo-av` (deprecated SDK 55) |
| Platform detection | `process.env.EXPO_OS` | `Platform.OS` in shared code |
| Safe area | `react-native-safe-area-context` | `SafeAreaView` from RN |
| Icons (SDK 55+) | `expo-symbols` | `@expo/vector-icons` |

## Responsiveness

Wrap root screens in `ScrollView` with `contentInsetAdjustmentBehavior="automatic"` — this handles notch/island safe area on iOS automatically:

```tsx
import { ScrollView } from 'react-native';

export default function Screen() {
  return (
    <ScrollView contentInsetAdjustmentBehavior="automatic">
      {/* content */}
    </ScrollView>
  );
}
```

Use `useWindowDimensions` for responsive sizing; avoid the `Dimensions` API (doesn't react to orientation changes):

```tsx
import { useWindowDimensions } from 'react-native';

const { width, height } = useWindowDimensions();
```

**Flexbox-first:** Prefer flexbox over absolute positioning. Use `gap` (flex gap) instead of margin for spacing between siblings.

**Shadows:** Use CSS `boxShadow` style (works on iOS and Android with New Architecture). Avoid `elevation` + `shadowColor` combo.

**Border radius:** Prefer `borderRadius` values that mirror the platform's "continuous corners" (iOS squircle): use 12–16 for cards, 8 for buttons.

## NativeTabs (SDK 55+)

NativeTabs renders platform-native tab bars (iOS 26 liquid glass, Android Material 3) — far smoother than the JS-based `@react-navigation/bottom-tabs`.

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router/ui';

export default function TabsLayout() {
  return (
    <Tabs>
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ focused }) => (
            <SymbolView name={focused ? 'house.fill' : 'house'} />
          ),
        }}
      />
      <Tabs.Screen
        name="settings"
        options={{
          title: 'Settings',
          tabBarIcon: ({ focused }) => (
            <SymbolView name={focused ? 'gear.circle.fill' : 'gear'} />
          ),
        }}
      />
    </Tabs>
  );
}
```

On **iOS 26+**, tabs render with the liquid glass material automatically. On **Android**, tabs follow Material Design 3. The JS-based tab implementations continue to work and are not removed in SDK 55.

## Animations (Reanimated)

**Reanimated v3 (stable):** Use for all production apps.

```tsx
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming,
  FadeIn,
  SlideInRight,
} from 'react-native-reanimated';

// Entering/exiting animations
<Animated.View entering={FadeIn.duration(300)} exiting={SlideInRight}>
  <Text>Hello</Text>
</Animated.View>

// Shared value + animated style
const opacity = useSharedValue(1);
const style = useAnimatedStyle(() => ({ opacity: opacity.value }));
// Trigger: opacity.value = withTiming(0, { duration: 200 });
```

**Reanimated v4 (opt-in, beta):** Experimental API with simplified syntax. Use only in new projects willing to track breaking changes:

```tsx
// v4 style (experimental)
import { animate, spring } from 'react-native-reanimated';
// API is still stabilizing — check release notes before adopting
```

**Rule of thumb:** v3 for production, v4 for prototyping / greenfield projects that can accept churn.

## Navigation Patterns

**Stack + Tabs:** Standard pattern for most apps.

```
app/
├── _layout.tsx          ← root Stack
├── (tabs)/
│   ├── _layout.tsx      ← Tabs layout
│   ├── index.tsx        ← home tab
│   └── profile.tsx      ← profile tab
└── modal.tsx            ← full-screen modal
```

**Link with previews (iOS):**

```tsx
import { Link } from 'expo-router';

<Link href="/product/42" preview={{ url: '/product/42/preview' }}>
  View Product
</Link>
```

**Context menus (iOS long-press):**

```tsx
<Link href="/item/1" contextMenu={{ items: [{ title: 'Share', systemImage: 'square.and.arrow.up' }] }}>
  Open Item
</Link>
```

**Haptics (iOS):**

```tsx
import * as Haptics from 'expo-haptics';
import { Platform } from 'react-native';

function onPress() {
  if (Platform.OS === 'ios') {
    Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);
  }
  // action...
}
```

## Key Points

- Never put non-route files in `app/` — they break Expo Router's file-routing
- `expo-image` has better caching, blurhash, and disk persistence vs RN `Image`
- Use `contentInsetAdjustmentBehavior="automatic"` on root `ScrollView` for safe-area handling
- NativeTabs (SDK 55+) gives iOS liquid glass and Android Material 3 for free
- Reanimated v3 is stable; v4 is beta — don't mix them in the same project
