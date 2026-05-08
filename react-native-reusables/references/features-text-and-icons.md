---
name: react-native-reusables-text-and-icons
description: Text component, inheritance (TextClassContext), and Icon wrapper usage.
---

# Text and icons

## Text

In React Native, styles do not cascade to children. The `Text` component is used for all visible text and supports an inheritance system so it can pick up styles from parent components (e.g. Button).

**TextClassContext:** Components like Button wrap their content with `TextClassContext`. The `Text` component reads from this context and applies the appropriate variant styles (e.g. button primary text color). Always use `Text` as the direct child of Button (or use the documented pattern) so the correct styles apply.

```tsx
import { Text } from '@/components/ui/text';

<Text>Hello, world!</Text>

// Inside Button — Text gets button variant styles via context
<Button variant="outline">
  <Text>Label</Text>
</Button>
```

Typography variants are available via props/className as documented in the Text component page.

## Icons

React Native Reusables uses a single **Icon** wrapper with Lucide icons instead of wrapping each icon. Import the wrapper and pass the icon component via `as`:

```tsx
import { Icon } from '@/components/ui/icon';
import { ChevronLeft } from 'lucide-react-native';

<Icon as={ChevronLeft} size={20} className="text-foreground" />
```

Use with Button for icon-only or icon+label buttons; keep `Text` for the label when the button has text so the inheritance system works.

## Key Points

- Use `Text` inside Button (and similar components) so `TextClassContext` can apply variant styles.
- Use `<Icon as={LucideIcon} />` for icons; do not wrap every Lucide icon individually.

<!--
Source references:
- https://github.com/founded-labs/react-native-reusables (docs: components/text.mdx, index.mdx)
- https://reactnativereusables.com/docs/components/text
- https://lucide.dev/guide/packages/lucide-react-native
-->
