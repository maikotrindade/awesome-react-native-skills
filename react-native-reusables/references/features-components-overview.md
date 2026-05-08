---
name: react-native-reusables-components-overview
description: How to use React Native Reusables UI components (Button, Input, Dialog, Text, variants, asChild).
---

# Components overview

Components are added via the CLI (`add <name>`) or copied manually from the registry. They live in your repo (e.g. `@/components/ui/`) and support Nativewind or Uniwind.

## Adding components

```bash
npx react-native-reusables add button input dialog text
```

Each component may require extra dependencies (e.g. `@rn-primitives/dialog` for Dialog). The CLI installs them when you add the component; for manual copy, check the component doc for the install step.

## Usage

**Button:** Use `Text` as direct child so the inheritance system can apply button text styles. Use `buttonVariants` and `buttonTextVariants` for link-as-button styling, or `asChild` with a Link.

```tsx
import { Button, buttonVariants, buttonTextVariants } from '@/components/ui/button';
import { Text } from '@/components/ui/text';

<Button variant="outline" size="sm">
  <Text>Label</Text>
</Button>

// Link that looks like a button
<Link className={buttonVariants({ variant: 'outline' })}>
  <Text className={buttonTextVariants({ variant: 'outline' })}>Click</Text>
</Link>

<Link href="/login" asChild>
  <Button><Text>Login</Text></Button>
</Link>
```

**Input / form controls:** Import from `@/components/ui/input`, etc.

```tsx
import { Input } from '@/components/ui/input';
<Input />
```

**Dialog / AlertDialog:** Require `PortalHost` in the root layout. Use compound components: `Dialog`, `DialogTrigger`, `DialogContent`, `DialogHeader`, `DialogTitle`, `DialogDescription`, (and `DialogFooter` for AlertDialog).

```tsx
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogDescription, DialogTrigger } from '@/components/ui/dialog';
import { Text } from '@/components/ui/text';

<Dialog>
  <DialogTrigger><Text>Open</Text></DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Title</DialogTitle>
      <DialogDescription>Description.</DialogDescription>
    </DialogHeader>
  </DialogContent>
</Dialog>
```

**Text:** Use for all visible text. Supports typography variants and inherits from `TextClassContext` when used inside components like Button.

```tsx
import { Text } from '@/components/ui/text';
<Text>Hello</Text>
```

## Key Points

- Components use `className` and often expose variants (e.g. `variant="outline"`, `size="sm"`). Use the `cn()` helper to merge class names.
- Overlay components (Dialog, DropdownMenu, Tooltip, Popover) require `PortalHost` in the root layout.

<!--
Source references:
- https://github.com/founded-labs/react-native-reusables (docs: components/button.mdx, input.mdx, dialog.mdx, text.mdx, alert-dialog.mdx)
- https://reactnativereusables.com/docs/components
-->
