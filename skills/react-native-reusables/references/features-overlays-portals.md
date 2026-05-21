---
name: react-native-reusables-overlays-portals
description: PortalHost requirement, overlay components (Dialog, Popover, Select), contentInsets for safe area.
---

# Overlays & Portals

Overlay components (Dialog, Popover, Select content, Dropdown, etc.) render through React Native's portal system. A **PortalHost** must be mounted at the root of the app for these to display correctly; otherwise overlay content may not appear or may be clipped.

## Usage

**PortalHost at root:** Place exactly one `PortalHost` at the end of the root layout, after all other providers and navigators:

```tsx
import { PortalHost } from '@rn-primitives/portal';

// app/_layout.tsx (Expo Router v4)
export default function RootLayout() {
  return (
    <ThemeProvider value={NAV_THEME[colorScheme]}>
      <Stack />
      <PortalHost />   {/* must be last */}
    </ThemeProvider>
  );
}
```

**Overlay components:** Dialog, Popover, SelectContent, DropdownMenuContent, and Tooltip render their content into the portal. Import from `@/components/ui/<component>` after adding via CLI.

**SelectContent insets:** Pass `contentInsets` so the content respects the safe area and keyboard. Use `useSafeAreaInsets()`:

```tsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';
import { Select, SelectTrigger, SelectValue, SelectContent, SelectItem } from '@/components/ui/select';

function MySelect() {
  const insets = useSafeAreaInsets();
  const contentInsets = { top: insets.top, bottom: insets.bottom, left: 12, right: 12 };

  return (
    <Select>
      <SelectTrigger className="w-[180px]">
        <SelectValue placeholder="Select..." />
      </SelectTrigger>
      <SelectContent insets={contentInsets} className="w-[180px]">
        <SelectItem label="Option" value="opt">Option</SelectItem>
      </SelectContent>
    </Select>
  );
}
```

**Dialog example:**

```tsx
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogDescription, DialogTrigger } from '@/components/ui/dialog';
import { Text } from '@/components/ui/text';

<Dialog>
  <DialogTrigger asChild>
    <Button><Text>Open</Text></Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Are you sure?</DialogTitle>
      <DialogDescription>This action cannot be undone.</DialogDescription>
    </DialogHeader>
  </DialogContent>
</Dialog>
```

**Programmatic control (DropdownMenu / components without open prop):**

Some components (e.g. `DropdownMenu`) cannot be controlled via `open`/`onOpenChange`. Use a `ref` to open/close programmatically after layout:

```tsx
const triggerRef = useRef<PressableRef>(null);

// Open dropdown from external button
<DropdownMenuTrigger ref={triggerRef}>
  <Text>Menu</Text>
</DropdownMenuTrigger>
```

**Dependencies:** Overlay primitives require the corresponding `@rn-primitives/*` package (e.g. `@rn-primitives/dialog`, `@rn-primitives/select`). The CLI `add` command installs them automatically.

## Key Points

- Without a root `PortalHost`, overlay content may not show or may be mispositioned — add it **once** at the very end of the root layout
- Use `contentInsets` (from `useSafeAreaInsets`) on SelectContent and similar so content is not hidden by notches or the keyboard
- `PortalHost` must come after all navigators and providers so overlays render on top of everything
