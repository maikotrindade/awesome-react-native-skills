---
name: react-native-reusables-installation
description: How to install React Native Reusables (CLI init vs manual setup) with NativeWind v4, Expo SDK 52+, and Expo Router v4.
---

# Installation

> Requires **NativeWind v4** (`nativewind@^4`), **Expo SDK 52+**, and **Expo Router v4**.

## Create project (CLI)

Scaffold a new Expo project with everything pre-configured:

```bash
npx react-native-reusables init
```

Options: `-c, --cwd <cwd>`, `-t, --template <template>`. Templates:

| Template | Description |
|----------|-------------|
| `minimal` | Bare Expo + NativeWind v4 + RNR |
| `minimal-uniwind` | Bare Expo + Uniwind + RNR |
| `clerk-auth` | Minimal + Clerk authentication pre-wired |

## Add components

After init (or after manual setup), add components by name:

```bash
npx react-native-reusables add button
npx react-native-reusables add input dialog
npx react-native-reusables add -a   # all components
```

Options: `-y` skip confirmation, `-o` overwrite existing files, `-p, --path <path>`.

Import from your configured aliases:

```tsx
import { Button } from '@/components/ui/button';
import { Text } from '@/components/ui/text';

export default function Screen() {
  return (
    <Button>
      <Text>Click me</Text>
    </Button>
  );
}
```

## Manual setup (existing project)

If you are not using `init`, you must:

### 1. NativeWind v4

Follow the [NativeWind v4 installation guide](https://www.nativewind.dev/docs/getting-started/installation). In `metro.config.js`, set `inlineRem: 16`:

```js
const { withNativeWind } = require('nativewind/metro');
module.exports = withNativeWind(config, { input: './global.css', inlineRem: 16 });
```

### 2. Dependencies

```bash
npx expo install tailwindcss-animate class-variance-authority clsx tailwind-merge @rn-primitives/portal
```

### 3. PortalHost

Render `<PortalHost />` from `@rn-primitives/portal` in the **root layout** as the **last child** of your providers. Required for Dialog, DropdownMenu, Tooltip, Popover, and any overlay:

```tsx
// app/_layout.tsx (Expo Router v4)
import { PortalHost } from '@rn-primitives/portal';

export default function RootLayout() {
  return (
    <ThemeProvider value={NAV_THEME[colorScheme]}>
      <Stack />
      <PortalHost />   {/* must be last */}
    </ThemeProvider>
  );
}
```

### 4. Path aliases

Configure in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@/*": ["./*"] }
  }
}
```

Also add to `babel.config.js` (or `app.config.js` for Expo):

```js
plugins: [['module-resolver', { alias: { '@': './' } }]]
```

### 5. Styles

Add CSS variables to `global.css` for `:root` and `.dark:root` (NativeWind v4 uses `.dark:root`, **not** `.dark`). Map them in `tailwind.config.js` and mirror values in `lib/theme.ts` (including `NAV_THEME` for React Navigation).

### 6. cn helper

Add `lib/utils.ts`:

```ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### 7. components.json

Create so the CLI knows paths and style variant:

```json
{
  "style": "new-york",
  "tailwind": { "config": "tailwind.config.ts", "css": "global.css", "baseColor": "zinc" },
  "aliases": { "components": "@/components", "utils": "@/lib/utils", "ui": "@/components/ui", "lib": "@/lib" }
}
```

### 8. Verify

```bash
npx react-native-reusables doctor
```

## Key Points

- Use **init** for new projects; use manual steps when adding to an existing Expo + NativeWind v4 app
- **NativeWind v4** requires `inlineRem: 16` in metro config and `.dark:root` (not `.dark`) for dark theme variables
- **PortalHost** is required for any overlay component — place it once at the end of the root layout
- Keep `global.css`, `tailwind.config`, and `theme.ts` in sync when changing theme variables

<!--
Source references:
- https://github.com/founded-labs/react-native-reusables (docs: installation/index.mdx, installation/manual.mdx)
- https://reactnativereusables.com/docs/installation
-->
