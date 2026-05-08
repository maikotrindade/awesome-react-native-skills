---
name: react-native-reusables-adding-components
description: Best practices for adding and integrating components (CLI vs manual, PortalHost, aliases, cn).
---

# Adding components — best practices

## Prefer the CLI

Use `npx react-native-reusables add <component>` so the correct files and dependencies (e.g. `@rn-primitives/dialog`) are installed and paths match your `components.json`. Use `-o` to overwrite when updating.

## PortalHost placement

Any overlay component (Dialog, AlertDialog, DropdownMenu, Tooltip, Popover, etc.) needs a mounted `PortalHost` from `@rn-primitives/portal`. Place it once in the root layout as the **last child** of your providers (e.g. after ThemeProvider and Stack). Without it, overlays may not render or may render in the wrong place.

```tsx
// app/_layout.tsx
<ThemeProvider value={NAV_THEME[colorScheme]}>
  <StatusBar style={...} />
  <Stack />
  <PortalHost />
</ThemeProvider>
```

## Path aliases

Ensure `tsconfig.json` has the same alias as in `components.json` (e.g. `"@/*": ["./*"]`) so imports like `@/components/ui/button` resolve. The CLI and docs assume `@/`; if you use something else, configure it in both places.

## cn helper

All components expect a `cn()` helper that merges class names (e.g. `clsx` + `tailwind-merge`). Provide it at the path specified in `components.json` (e.g. `@/lib/utils`):

```ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

## Manual copy

If you copy components manually:

1. Install any primitives the component doc lists (e.g. `npx expo install @rn-primitives/dialog`).
2. Copy from the correct styling variant (`nativewind` vs `uniwind`) in the registry to match your project.
3. Update import paths to your aliases (e.g. `@/lib/utils`, `@/components/ui/text`).
4. Add `PortalHost` to the root layout if the component uses Portal.

## Verifying setup

Run `npx react-native-reusables doctor` after setup or when something breaks. Use `--log-level all` before the command for detailed logs when reporting issues.

## Key Points

- PortalHost is required for Dialog, DropdownMenu, Tooltip, Popover, AlertDialog—place it in root layout.
- Keep path aliases and `components.json` in sync; provide `cn` at the configured utils path.

<!--
Source references:
- https://github.com/founded-labs/react-native-reusables (docs: installation/manual.mdx, cli.mdx, components/*.mdx)
- https://reactnativereusables.com/docs/installation
-->
