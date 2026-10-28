---
name: react-native-i18n-rtl
description: Right-to-left support in React Native — I18nManager, Expo supportsRTL/forcesRTL, logical start/end styles, icon and animation mirroring, text alignment, and reloading after a direction change.
---

# RTL (Right-to-Left)

Arabic, Hebrew, Persian and Urdu read right to left. With RTL enabled, React Native mirrors flexbox: `flexDirection: 'row'` starts on the right, and `start`/`end` swap sides.

## Enable RTL

**Expo** — expo-localization config plugin:

```json
{
  "expo": {
    "plugins": [
      ["expo-localization", { "supportsRTL": true, "forcesRTL": false }]
    ]
  }
}
```

`supportsRTL` lets the OS switch the app to RTL when the device language is RTL (it sets `android:supportsRtl` and the iOS flag). `forcesRTL` is for testing. Rebuild the native app after changing the plugin.

**Bare React Native:**

```ts
import { I18nManager } from 'react-native';

I18nManager.allowRTL(true); // follow the device direction
```

Android also needs `android:supportsRtl="true"` on `<application>` in `AndroidManifest.xml`.

## Switching direction from an in-app language picker

The layout direction is read when the JS bundle starts. Changing it requires a reload:

```ts
import { I18nManager } from 'react-native';
import * as Updates from 'expo-updates'; // bare: react-native-restart

export async function applyLanguage(lng: string) {
  await i18n.changeLanguage(lng);
  storage.set('language', lng);

  const shouldBeRTL = i18n.dir(lng) === 'rtl';
  if (I18nManager.isRTL !== shouldBeRTL) {
    I18nManager.allowRTL(shouldBeRTL);
    I18nManager.forceRTL(shouldBeRTL);
    await Updates.reloadAsync(); // persisted direction applies on next start
  }
}
```

`Updates.reloadAsync()` needs `expo-updates` installed and doesn't work in Expo Go; in development use `DevSettings.reload()`.

## Styles: use logical properties

| Avoid | Use |
|-------|-----|
| `marginLeft` / `marginRight` | `marginStart` / `marginEnd` |
| `paddingLeft` / `paddingRight` | `paddingStart` / `paddingEnd` |
| `left` / `right` | `start` / `end` |
| `borderLeftWidth`, `borderTopLeftRadius` | `borderStartWidth`, `borderTopStartRadius` |
| `textAlign: 'left'` | `textAlign: 'auto'` (default) or omit |

```ts
const styles = StyleSheet.create({
  row: { flexDirection: 'row', alignItems: 'center' }, // mirrors automatically
  avatar: { marginEnd: 12 },
  badge: { position: 'absolute', top: 4, end: 4 },
});
```

NativeWind / Tailwind: use `ms-*`, `me-*`, `ps-*`, `pe-*`, `start-*`, `end-*`, and `rtl:` / `ltr:` variants.

`flexDirection: 'row-reverse'` reverses *again* in RTL — use it only when the reversed order is intentional in both directions.

## What to mirror and what not to

| Mirror | Don't mirror |
|--------|--------------|
| Back/forward arrows, chevrons | Media playback controls (play, fast-forward) |
| Progress direction, sliders | Clocks, charts with numeric x-axes |
| Swipe-to-delete direction, drawers | Logos, photos, checkmarks |
| List bullets, navigation order | Phone numbers, code, math |

```tsx
<ChevronIcon style={{ transform: [{ scaleX: I18nManager.isRTL ? -1 : 1 }] }} />
```

Animations: invert horizontal translations (`translateX: I18nManager.isRTL ? -x : x`) and gesture thresholds. React Navigation and Expo Router stacks already slide from the correct side.

## Text

- Leave `textAlign` unset: text aligns to the start of its own direction.
- Mixed-direction strings (an English product name inside Arabic text) are handled by the Unicode bidi algorithm; for numbers or phone numbers that render in the wrong order, wrap them with LRM/RLM marks (`‎`, `‏`) or isolate characters (`⁦…⁩`).
- `TextInput` follows the keyboard language; set `textAlign` explicitly only for always-LTR fields (email, URL, code).

## Testing

- Force RTL in development (`forcesRTL: true` or `I18nManager.forceRTL(true)` + reload) and click through every screen.
- iOS: Scheme → Run → Options → App Language → "Right-to-Left Pseudolanguage".
- Android: Developer options → "Force RTL layout direction".
- Add an RTL locale to E2E flows (see `react-native-e2e-testing/features-advanced-flows.md`) so regressions are caught.

## Key Points

- Enable RTL support natively (`supportsRTL` / `allowRTL` + `android:supportsRtl`).
- Direction changes apply after a reload; persist the choice and reload once.
- Use `start`/`end` everywhere; mirror directional icons and horizontal animations, not media or logos.
- Test with forced RTL on both platforms.
