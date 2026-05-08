---
name: react-native-reusables-customization
description: Theme and style customization (components.json, global.css, tailwind.config, theme.ts).
---

# Customization

React Native Reusables uses a shadcn/ui-style theme system with four main pieces: `components.json`, `global.css`, `tailwind.config.ts` (or `.js`), and `theme.ts`.

## components.json

Configures where the CLI scaffolds files (paths, style). Same role as [shadcn/ui components.json](https://ui.shadcn.com/docs/components-json). Only change it when switching styles or paths.

## global.css

Defines light and dark theme via CSS variables:

- `:root` — light theme
- `.dark:root` — dark theme (use `.dark:root` not `.dark` for Nativewind)

Tailwind classes like `bg-background`, `text-foreground` resolve to these variables. You can start from [shadcn/ui themes](https://ui.shadcn.com/themes) (Tailwind v3 / CSS variables) and replace `.dark` with `.dark:root`.

## tailwind.config

Connects utilities to CSS variables (e.g. `background: 'hsl(var(--background))'`), sets `darkMode: 'class'`, and can add `tailwindcss-animate` and Nativewind-specific keys (e.g. `hairlineWidth`, accordion keyframes).

## theme.ts

Exports the same colors as in `global.css` as a TypeScript object (e.g. `THEME.light`, `THEME.dark`) for use in JS logic, inline styles, or animations. Also exports `NAV_THEME` for React Navigation's `ThemeProvider` in `_layout.tsx`.

When you change a CSS variable in `global.css`, update the matching value in `theme.ts`. Sync prompt:

> Read CSS variables under `:root` and `.dark:root` in `global.css`. Update the `light` and `dark` entries in the `THEME` object in `theme.ts` to match in HSL. Keep keys and `NAV_THEME` unchanged; add new variables if missing; comment stale ones.

## Key Points

- Edit `global.css` for colors/radius; then update `tailwind.config` if you add new variables, and keep `theme.ts` in sync.
- Use `THEME` and `NAV_THEME` from `theme.ts` for navigation and any non-class styling.

<!--
Source references:
- https://github.com/founded-labs/react-native-reusables (docs: customization.mdx)
- https://reactnativereusables.com/docs/customization
-->
