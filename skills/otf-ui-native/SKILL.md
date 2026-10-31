---
name: otf-ui-native
description: Build Expo and React Native interfaces with the free MIT @otfdashkit/ui-native component library. Use when an app needs shared component names and design tokens with OTF's web SDK, or when setting up OTFProvider.
metadata:
  author: dave-otf
  version: "1.0.0"
---

# OTF UI Native

Use this skill when building an Expo or React Native interface with `@otfdashkit/ui-native`.

## Install

```bash
pnpm add @otfdashkit/ui-native @otfdashkit/tokens
npx expo install react-native-svg react-native-safe-area-context
```

Check the [package README](https://github.com/otf-kit/sdk/tree/main/packages/ui-native) for the current peer dependencies and install any that your app is missing.

## Set up the provider

For an Expo Router app, wrap the root layout once:

```tsx
import { OTFProvider } from '@otfdashkit/ui-native'

export default function RootLayout() {
  return (
    <OTFProvider>
      {/* navigator and screens */}
    </OTFProvider>
  )
}
```

## Use components

```tsx
import { Button, Card, Input } from '@otfdashkit/ui-native'
```

The native package shares component names and design tokens with `@otfdashkit/ui` on web. Props can differ between platforms, so check the native component page before copying a web example.

## References

- [Getting started](references/getting-started.md)
- [OTF component gallery and docs](https://otf-kit.dev/components?utm_source=directory&utm_medium=organic&utm_campaign=awesome-rn-skills&utm_content=2026-10-31-awesome-rn-skills-ui-native)
- [Native showcase](https://native-preview.otf-kit.dev/)
- [Package source](https://github.com/otf-kit/sdk/tree/main/packages/ui-native)
- [npm package](https://www.npmjs.com/package/@otfdashkit/ui-native)
