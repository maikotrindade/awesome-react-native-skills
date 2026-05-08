---
name: react-native-reusables-blocks
description: Pre-built blocks (e.g. authentication) and how to add them with the CLI.
---

# Blocks

Blocks are pre-built sections (e.g. auth forms, user menu) that you can add with the CLI or copy from the docs. They are composed of React Native Reusables components.

## Authentication blocks

The docs list blocks such as:

- Sign-in form
- Sign-up form
- Forgot password form
- Reset password form
- Verify email form
- User menu
- Social connections

Add them the same way as components: `npx react-native-reusables add <block-name>` (or use the block names listed in the docs). Paths and dependencies follow the same rules as components.

## Clerk integration

Authentication blocks can be wired to [Clerk](https://clerk.com). Use the integration option in the docs to get blocks that use Clerk. For a new project with Clerk auth pre-configured, use the `clerk-auth` template:

```bash
npx react-native-reusables init -t clerk-auth
```

After adding your first Clerk block, complete setup with the [Expo quick start](https://clerk.com/docs/quickstarts/expo) (API keys, etc.).

## Key Points

- Prefer **add** via CLI so dependencies and files are aligned with the registry.
- Blocks live in your app; customize by editing the added files and keeping imports (e.g. `@/components/ui/*`) consistent with your `components.json` aliases.

<!--
Source references:
- https://github.com/founded-labs/react-native-reusables (docs: blocks/authentication/index.mdx)
- https://reactnativereusables.com/docs/blocks
-->
