---
name: react-native-reusables
description: shadcn/ui-style components for React Native with NativeWind v4, RN Primitives, and CLI scaffolding. Use when building Expo apps with copy-paste UI components, theming, or overlay patterns.
metadata:
  author: maikotrindade
  version: "2026.5.0"
  source: Reviewed and updated from https://github.com/hairyf/skills/tree/main/skills/react-native-reusables
---

# React Native Reusables

> Targets NativeWind v4, Expo SDK 52+, and Expo Router v4. For Expo SDK tooling see the `react-native-expo` skill. For standalone state/navigation patterns see the `react-native-ecosystem` skill.

React Native Reusables brings the shadcn/ui experience to React Native: copy-paste or CLI-scaffolded components, theming via CSS variables, and RN Primitives under the hood. It supports NativeWind and Uniwind, requires a root `PortalHost` for overlays, and uses a `Text` inheritance system and an `Icon` wrapper for Lucide.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Overview | What RNR is, differences from shadcn/ui (portals, no cascade, icons, etc.) | [core-overview](references/core-overview.md) |
| Installation | CLI init vs manual setup, adding components, PortalHost, dependencies | [core-installation](references/core-installation.md) |
| Customization | components.json, global.css, tailwind.config, theme.ts | [core-customization](references/core-customization.md) |

## Features

### Components and patterns

| Topic | Description | Reference |
|-------|-------------|-----------|
| Components overview | Button, Input, Dialog, variants, asChild, compound components | [features-components-overview](references/features-components-overview.md) |
| Text and icons | TextClassContext inheritance, Icon wrapper with Lucide | [features-text-and-icons](references/features-text-and-icons.md) |
| Forms & controls | Label, Input, Select, Checkbox, RadioGroup, react-hook-form integration | [features-forms-controls](references/features-forms-controls.md) |
| Overlays & portals | PortalHost, Dialog/Popover/SelectContent, contentInsets | [features-overlays-portals](references/features-overlays-portals.md) |
| Registry and CLI | init, add, doctor; custom registry and registryDependencies | [features-registry-cli](references/features-registry-cli.md) |
| Blocks | Auth blocks, Clerk integration, adding blocks via CLI | [features-blocks](references/features-blocks.md) |

### Best practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| Adding components | Prefer CLI, PortalHost placement, path aliases, cn helper, manual copy | [best-practices-adding-components](references/best-practices-adding-components.md) |
| Troubleshooting | doctor, --log-level all, PortalHost, aliases, dependencies | [best-practices-troubleshooting](references/best-practices-troubleshooting.md) |

## Key Recommendations

- **PortalHost is required** for any overlay (Dialog, DropdownMenu, Popover, Tooltip) — place it last in the root layout
- **Use the CLI** (`npx react-native-reusables add <name>`) so primitives and paths are wired correctly
- **Text inside Button** — always use `<Text>` from `@/components/ui/text` as the direct child so `TextClassContext` applies variant styles
- **Icons** — use `<Icon as={LucideIcon} />` rather than wrapping each icon individually
- **NativeWind v4** — use `.dark:root` (not `.dark`) in `global.css` for dark-mode CSS variables
- **Run `doctor`** after manual setup or when something breaks before assuming a bug
