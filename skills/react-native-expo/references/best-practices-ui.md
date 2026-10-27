---
name: react-native-expo-best-practices-ui
description: Building native Expo UI — code style, expo-image, responsiveness, NativeTabs (SDK 55+), Link previews/menus, and Reanimated 4.
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

NativeTabs renders the platform's own tab bar (iOS Liquid Glass, Android Material 3) instead of a JS-drawn one.

```tsx
// app/(tabs)/_layout.tsx
import { NativeTabs } from 'expo-router/native-tabs'; // SDK 55–57: 'expo-router/unstable-native-tabs'

export default function TabsLayout() {
  return (
    <NativeTabs minimizeBehavior="onScrollDown">
      <NativeTabs.Trigger name="index">
        <NativeTabs.Trigger.Icon sf="house.fill" md="home" />
        <NativeTabs.Trigger.Label>Home</NativeTabs.Trigger.Label>
      </NativeTabs.Trigger>
      <NativeTabs.Trigger name="inbox">
        <NativeTabs.Trigger.Icon sf="tray.fill" md="inbox" />
        <NativeTabs.Trigger.Label>Inbox</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Badge>3</NativeTabs.Trigger.Badge>
      </NativeTabs.Trigger>
      <NativeTabs.Trigger name="(search)" role="search">
        <NativeTabs.Trigger.Label>Search</NativeTabs.Trigger.Label>
      </NativeTabs.Trigger>
    </NativeTabs>
  );
}
```

- `sf` is an SF Symbol name (iOS); `md` is a Material Symbols name (Android). On SDK 58, Android `md` icons need `expo-symbols` installed.
- Each `name` must match a route file in the same folder.
- The JS-based `Tabs` from `expo-router` still works when you need fully custom tab bar UI.

## Animations (Reanimated)

Expo SDK 54 and later ship Reanimated 4, which needs `react-native-worklets` (`npx expo install react-native-reanimated react-native-worklets`). `babel-preset-expo` configures the Babel plugin, so no `babel.config.js` change is needed.

```tsx
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';

// State-driven: CSS transition, no worklets
<Animated.View
  style={{
    opacity: visible ? 1 : 0,
    transitionProperty: 'opacity',
    transitionDuration: 200,
  }}
/>

// Mount / unmount
<Animated.View entering={FadeIn.duration(300)} exiting={FadeOut}>
  <Text>Hello</Text>
</Animated.View>
```

Use shared values (`useSharedValue` + `useAnimatedStyle`) for gesture- or scroll-driven motion. See `react-native-ecosystem/features-reanimated.md` for the full API and v3 → v4 migration.

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

**Link previews and context menus (iOS long-press):**

```tsx
import { Link } from 'expo-router';

<Link href="/product/42" asChild>
  <Link.Trigger>
    <Pressable>
      <ProductCard id={42} />
    </Pressable>
  </Link.Trigger>
  <Link.Preview />
  <Link.Menu>
    <Link.MenuAction title="Share" icon="square.and.arrow.up" onPress={share} />
    <Link.MenuAction title="Delete" icon="trash" destructive onPress={remove} />
  </Link.Menu>
</Link>
```

`<Link.Preview />` renders the destination route as a peek preview; `icon` values are SF Symbol names.

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
- NativeTabs (SDK 55+) gives iOS Liquid Glass and Android Material 3 for free; import from `expo-router/native-tabs` on SDK 58
- Reanimated 4 is the default on current SDKs; prefer CSS transitions for state-driven animation
- `ImageBackground` is deprecated in SDK 58 — use a `View` with an absolutely positioned `expo-image` `Image`
