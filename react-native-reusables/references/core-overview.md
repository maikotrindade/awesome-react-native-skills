---
name: react-native-reusables-overview
description: What React Native Reusables is and how it differs from shadcn/ui on the web.
---

# Overview

React Native Reusables is not a component library—it is a system for building your own component library. It brings the shadcn/ui experience to React Native using Nativewind or Uniwind, RN Primitives, and consistent naming so you can build universal apps with a familiar design system.

## Key differences from shadcn/ui

- **Styling:** Uses [Nativewind](https://www.nativewind.dev) or [Uniwind](https://www.uniwind.dev) for Tailwind-like classes in React Native. Components are scaffolded into your app (Nativewind or Uniwind variants).
- **Primitives:** Built on [RN Primitives](https://rnprimitives.com)—a React Native port of Radix UI with a similar API for composition.
- **Portals:** React Native has no DOM portals. Modals, menus, tooltips, etc. render into a [`PortalHost`](https://rnprimitives.com/portal/) from `@rn-primitives/portal`. You must mount `<PortalHost />` in your root layout (e.g. last child of providers).
- **No cascading styles:** Children (e.g. `Text`) do not inherit parent styles. Each element is styled explicitly. The project uses a `TextClassContext` workaround so `Text` inside components like `Button` can pick up the correct variant styles.
- **No data attributes:** No `data-*` attributes in RN; variants are driven by props/state.
- **Animations:** Uses `react-native-reanimated` for animations.
- **Icons:** Uses a wrapper component with [Lucide React Native](https://lucide.dev/guide/packages/lucide-react-native), e.g. `<Icon as={LeftArrowIcon} />`, instead of wrapping each icon.
- **Programmatic control:** Some components (e.g. `DropdownMenu`) cannot be controlled via `open` / `onOpenChange`; use a `ref` to open/close after layout.

## Usage

- Add components with the CLI (`npx react-native-reusables add <name>`) or copy from the registry manually.
- Import from your aliased paths (e.g. `@/components/ui/button`, `@/components/ui/text`).
- Use the official docs for [installation](/docs/installation), [customization](/docs/customization), and [CLI](/docs/cli).

<!--
Source references:
- https://github.com/founded-labs/react-native-reusables (docs: index.mdx)
- https://reactnativereusables.com/docs
-->
