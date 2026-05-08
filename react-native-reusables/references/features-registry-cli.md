---
name: react-native-reusables-registry-cli
description: CLI commands (init, add, doctor) and using or creating a custom registry.
---

# Registry and CLI

The CLI is the main way to add components and check project setup. It uses the shadcn CLI under the hood.

## Commands

**init** — Scaffold a new Expo project:

```bash
npx react-native-reusables init [options]
# -c, --cwd <cwd>   working directory
# -t, --template <template>   minimal | minimal-uniwind | clerk-auth
```

**add** — Add one or more components:

```bash
npx react-native-reusables add [options] [...components]
# -c, --cwd <cwd>
# -y, --yes         skip confirmation
# -o, --overwrite   overwrite existing files
# -a, --all         add all components
# -p, --path <path>
```

**doctor** — Check setup and diagnose issues:

```bash
npx react-native-reusables doctor [options]
# -c, --cwd <cwd>
# -s, --summary     summary only
# -y, --yes         skip confirmation
```

Debug with: `npx react-native-reusables --log-level all <command> [...]`

## Create your own registry

To host your own registry (so others can add your components):

1. Follow the [shadcn/ui registry guide](https://ui.shadcn.com/docs/registry).
2. Optional: use the [rnr-registry-template](https://github.com/gabimoncha/rnr-registry-template) for React Native Reusables.

**Using React Native Reusables in your registry:** In each [`registry-item.json`](https://ui.shadcn.com/docs/registry/registry-item-json), add React Native Reusables as `registryDependencies` with the full registry item URL:

```
http://reactnativereusables.com/r/new-york/<COMPONENT_NAME>.json
```

Replace `<COMPONENT_NAME>` with the component name (e.g. `button`, `dialog`).

**Installing from a custom registry:** Use the full URL with `add`:

```bash
npx react-native-reusables add http://localhost:3000/r/hello-world.json
# or
npx shadcn add http://localhost:3000/r/hello-world.json
```

## Key Points

- Use `doctor` after setup or when debugging; use `--log-level all` for verbose output when filing bugs.
- Custom registries list RNR components via full URLs in `registryDependencies`.

<!--
Source references:
- https://github.com/founded-labs/react-native-reusables (docs: cli.mdx, create-your-own-registry.mdx)
- https://reactnativereusables.com/docs/cli
- https://reactnativereusables.com/docs/create-your-own-registry
-->
