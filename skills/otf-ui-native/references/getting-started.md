# Getting started with OTF UI Native

`@otfdashkit/ui-native` is a React Native component package for Expo apps. Its README lists 85+ components and states that the package supports iOS, Android, and web export.

## Install

```bash
pnpm add @otfdashkit/ui-native @otfdashkit/tokens
npx expo install react-native-svg react-native-safe-area-context
```

Check the current [package README](https://github.com/otf-kit/sdk/tree/main/packages/ui-native) for peer dependencies and component-specific setup.

## Wrap the app root

```tsx
import { OTFProvider } from '@otfdashkit/ui-native'

export default function RootLayout() {
  return <OTFProvider>{/* navigator and screens */}</OTFProvider>
}
```

## Render components

```tsx
import { Button, Card, Input } from '@otfdashkit/ui-native'
```

The package uses component names and design tokens that also appear in OTF's web SDK. Native and web props are not identical. Use the native component documentation when writing Expo code.

## Learn more

- [OTF component gallery and docs](https://otf-kit.dev/components?utm_source=directory&utm_medium=organic&utm_campaign=awesome-rn-skills&utm_content=2026-10-31-awesome-rn-skills-ui-native)
- [Native showcase](https://native-preview.otf-kit.dev/)
- [npm package](https://www.npmjs.com/package/@otfdashkit/ui-native)
