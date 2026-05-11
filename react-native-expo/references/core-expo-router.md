---
name: react-native-expo-router
description: Expo Router file-based routing — layouts, route groups, typed params, API routes, and SDK 56 migration from @react-navigation.
---

# Expo Router

Expo Router is a file-based router for React Native and web built on React Navigation. Every file in the `app/` directory becomes a route automatically.

## Usage

**Route files:** Create a file in `app/` to define a route:

```
app/
├── _layout.tsx          ← root layout (required)
├── index.tsx            ← "/" route
├── profile.tsx          ← "/profile"
├── (tabs)/              ← route group (no URL segment)
│   ├── _layout.tsx
│   ├── home.tsx
│   └── settings.tsx
└── user/
    └── [id].tsx         ← "/user/:id" (dynamic)
```

**Root layout** (`app/_layout.tsx`):

```tsx
import { Stack } from 'expo-router';
import { ThemeProvider } from '@react-navigation/native';

export default function RootLayout() {
  return (
    <ThemeProvider value={colorScheme === 'dark' ? DarkTheme : DefaultTheme}>
      <Stack />
    </ThemeProvider>
  );
}
```

**Typed params:** Use `useLocalSearchParams` for dynamic routes:

```tsx
import { useLocalSearchParams } from 'expo-router';

export default function UserScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  return <Text>User {id}</Text>;
}
```

**Navigation:** Use `Link` for declarative navigation, `router` for imperative:

```tsx
import { Link, router } from 'expo-router';

// Declarative
<Link href="/profile">Go to Profile</Link>
<Link href={{ pathname: '/user/[id]', params: { id: '42' } }}>Open User</Link>

// Imperative
router.push('/profile');
router.replace('/login');
router.back();
```

**Modals and sheets:**

```tsx
// app/_layout.tsx
<Stack>
  <Stack.Screen name="index" />
  <Stack.Screen name="modal" options={{ presentation: 'modal' }} />
</Stack>
```

## API Routes

API routes enable server-side logic without a separate backend. Use the `+api.ts` suffix:

```
app/
├── api/
│   ├── hello+api.ts     → GET /api/hello
│   └── user/[id]+api.ts → /api/user/:id
```

Export named HTTP handlers:

```ts
// app/api/hello+api.ts
import { ExpoRequest, ExpoResponse } from 'expo-router/server';

export async function GET(request: ExpoRequest): Promise<ExpoResponse> {
  return ExpoResponse.json({ message: 'Hello' });
}

export async function POST(request: ExpoRequest): Promise<ExpoResponse> {
  const body = await request.json();
  return ExpoResponse.json({ received: body });
}
```

**EAS Hosting runtime (Cloudflare Workers) limits:**
- No Node.js `fs` module — use Web APIs or cloud databases
- Supported databases: Cloudflare D1 (SQLite), Turso, PlanetScale, Supabase, Neon
- Secrets: `eas env:create` → available as `process.env.MY_SECRET` in API routes only
- Never expose secrets via `EXPO_PUBLIC_` prefix

**Local dev and deploy:**

```bash
npx expo serve          # local API route testing
eas deploy              # deploy to EAS Hosting
```

## SDK 56: Migrating from @react-navigation

SDK 56 requires redirecting `@react-navigation/*` imports to `expo-router/*` entry points. The runtime API is identical — only the import source changes.

**Automated codemod (preferred):**

```bash
npx expo-codemod sdk-56-expo-router-react-navigation-replace src
```

**Manual mapping:**

| Old import | New import |
|------------|------------|
| `@react-navigation/native` | `expo-router/react-navigation` |
| `@react-navigation/stack` | `expo-router/js-stack` |
| `@react-navigation/bottom-tabs` | `expo-router/js-tabs` |
| `@react-navigation/drawer` | `expo-router/js-drawer` |

**Critical:** Do NOT rewrite `import { Stack } from 'expo-router'` to `expo-router/js-stack`. The root `Stack` is Expo Router's layout component for route files and is different from the React Navigation stack.

**Post-migration validation:**
1. Search for any remaining `@react-navigation/` imports
2. Check for deprecated symbols in the new `expo-router/*` modules
3. Run `npx tsc --noEmit` and `npx expo start`
4. Remove unused `@react-navigation/*` packages from `package.json`

## Key Points

- Routes live exclusively in `app/`; never co-locate non-route code (components, utils, types) there
- A root route at `app/index.tsx` (the `/` path) is required
- API routes run on Cloudflare Workers on EAS Hosting — no Node.js filesystem APIs
- SDK 56+: use the codemod to migrate react-navigation imports; don't rewrite `Stack` from `expo-router`

<!--
Source references:
- https://github.com/expo/skills (plugins/expo/skills/expo-api-routes/SKILL.md)
- https://github.com/expo/skills (plugins/expo/skills/upgrading-expo/references/react-navigation-to-expo-router.md)
- https://docs.expo.dev/router/introduction/
-->
