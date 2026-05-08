---
name: react-native-reusables-troubleshooting
description: Troubleshooting React Native Reusables — doctor command, verbose logging, and fixes for common issues (PortalHost, aliases, missing dependencies).
---

# Troubleshooting & Debugging

Use the CLI `doctor` command to validate setup. When something fails, run with verbose logging to gather details for bug reports or local fixes.

## Usage

**doctor:** Check project setup and get actionable fixes:

```bash
npx react-native-reusables doctor [options]
# -c, --cwd <cwd>   working directory
# -s, --summary     summary only
# -y, --yes         skip confirmation
```

**Verbose logging:** Prepend `--log-level all` to any command for full output:

```bash
npx react-native-reusables --log-level all doctor
npx react-native-reusables --log-level all add button
```

## Common Issues

**Overlays not showing**
Ensure `PortalHost` from `@rn-primitives/portal` is mounted at the app root as the **last child** of providers. Without it, Dialog, Popover, SelectContent, etc. will not render or may be clipped behind other views.

```tsx
// app/_layout.tsx — correct placement
<ThemeProvider value={NAV_THEME[colorScheme]}>
  <Stack />
  <PortalHost />   {/* last */}
</ThemeProvider>
```

**Path aliases not resolving (`@/components/ui/...`)**
Configure `@/` in `tsconfig.json` (`"@/*": ["./*"]`) and in Metro/Babel. Run `doctor` to verify. If you use a different alias, update both `tsconfig.json` and `components.json`.

**Missing `@rn-primitives/*` packages**
If you copied components manually, install the required primitive:

```bash
npx expo install @rn-primitives/dialog @rn-primitives/select @rn-primitives/checkbox
```

The CLI `add` command installs these automatically — prefer it over manual copy.

**`Text` or `Icon` not found**
Many components import `Text` and `Icon` from `@/components/ui/text` and `@/components/ui/icon`. Add them first:

```bash
npx react-native-reusables add text icon
```

**NativeWind v4 styles not applying**
- Confirm `global.css` is imported in the entry file (`app/_layout.tsx`)
- Confirm `tailwind.config` has `content` pointing to your source files
- Dark mode: use `.dark:root` in `global.css`, **not** `.dark`
- Set `inlineRem: 16` in `withNativeWind` metro config

**`cn` not found / merge errors**
Ensure `lib/utils.ts` exports a `cn` function using `clsx` + `tailwind-merge` at the path specified in `components.json` (default `@/lib/utils`).

**react-hook-form Controller types**
If `Controller` renders an RNR `Input` and you see type errors, cast `onChangeText` → `onChange`:

```tsx
render={({ field: { onChange, onBlur, value } }) => (
  <Input onChangeText={onChange} onBlur={onBlur} value={value} />
)}
```

## Key Points

- Run `doctor` after init or when adding components manually; fix any reported issues before assuming a library bug
- When reporting issues, include the output of `npx react-native-reusables --log-level all doctor` and the failing command
- Overlay issues almost always come down to a missing or mis-placed `PortalHost`

<!--
Source references:
- https://github.com/founded-labs/react-native-reusables (apps/docs/content/docs/cli.mdx)
- https://reactnativereusables.com/docs/cli
-->
