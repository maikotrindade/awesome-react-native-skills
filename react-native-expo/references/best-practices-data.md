---
name: react-native-expo-best-practices-data
description: Data fetching in Expo — fetch vs React Query, expo-secure-store, offline support, environment variables, and Expo Router loaders.
---

# Data Fetching & Storage

## When to Use What

| Scenario | Approach |
|----------|----------|
| Simple one-off requests | `fetch` with typed error handling |
| Cached / deduped API calls | TanStack Query (`useQuery`, `useMutation`) |
| Route-level data loading (web, SDK 55+) | Expo Router loaders |
| Token / secrets | `expo-secure-store` |
| Non-sensitive persistent data | `AsyncStorage` or MMKV |

**Prefer `fetch` over `axios`** in Expo projects — it's built into Hermes and works in API routes (Cloudflare Workers), where axios has polyfill issues.

## fetch Patterns

**Typed error class:**

```ts
class APIError extends Error {
  constructor(public status: number, message: string) {
    super(message);
    this.name = 'APIError';
  }
}

async function apiFetch<T>(url: string, options?: RequestInit): Promise<T> {
  const response = await fetch(url, options);
  if (!response.ok) {
    throw new APIError(response.status, `HTTP ${response.status}`);
  }
  return response.json() as Promise<T>;
}
```

**Cancellation with AbortController:**

```ts
const controller = new AbortController();
const data = await apiFetch('/api/user', { signal: controller.signal });

// Cancel (e.g. on component unmount):
controller.abort();
```

## TanStack Query (React Query)

```bash
npx expo install @tanstack/react-query
```

```tsx
// Provider in app/_layout.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
const queryClient = new QueryClient();

<QueryClientProvider client={queryClient}>
  <Stack />
</QueryClientProvider>
```

**Query with staleTime:**

```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => apiFetch<User>(`/api/user/${userId}`),
  staleTime: 60_000,   // don't refetch if data is < 1 min old
});
```

**Mutation:**

```tsx
const mutation = useMutation({
  mutationFn: (newTodo: CreateTodoInput) =>
    apiFetch<Todo>('/api/todos', {
      method: 'POST',
      body: JSON.stringify(newTodo),
      headers: { 'Content-Type': 'application/json' },
    }),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
```

## Expo Router Loaders (Web + SDK 55+)

Route loaders run at the route level before the component renders — great for SSR/web data loading:

```ts
// app/user/[id].tsx
import { useLoader } from 'expo-router/server';

export async function loader({ params }: { params: { id: string } }) {
  const user = await fetchUser(params.id);
  return { user };
}

export default function UserScreen() {
  const { user } = useLoader<typeof loader>();
  return <Text>{user.name}</Text>;
}
```

Loaders currently work on web only; on native, the component falls back to standard data fetching.

## Secure Token Storage

**Always use `expo-secure-store` for tokens** — AsyncStorage is unencrypted and readable from the device file system.

```bash
npx expo install expo-secure-store
```

```ts
import * as SecureStore from 'expo-secure-store';

// Save
await SecureStore.setItemAsync('auth_token', token);

// Read
const token = await SecureStore.getItemAsync('auth_token');

// Delete
await SecureStore.deleteItemAsync('auth_token');
```

**Token refresh pattern:**

```ts
async function getAuthHeaders(): Promise<HeadersInit> {
  let token = await SecureStore.getItemAsync('auth_token');
  if (isExpired(token)) {
    token = await refreshToken();
    await SecureStore.setItemAsync('auth_token', token);
  }
  return { Authorization: `Bearer ${token}` };
}
```

## Environment Variables

```
EXPO_PUBLIC_API_URL=https://api.example.com   ← inlined at build time, visible in bundle
DATABASE_URL=postgres://...                    ← server-only, available in API routes
```

- `EXPO_PUBLIC_*` — for client-side config (API base URLs, feature flags). **Never use for secrets.** Values are embedded in the JS bundle.
- Non-prefixed vars — available only in API routes (`+api.ts` files) running on EAS Hosting.

```ts
// Client code
const API_URL = process.env.EXPO_PUBLIC_API_URL;

// API route (server-only)
export async function GET() {
  const db = process.env.DATABASE_URL; // never reaches the client
}
```

## Offline Support

```bash
npx expo install @react-native-community/netinfo
```

```ts
import NetInfo from '@react-native-community/netinfo';
import { onlineManager } from '@tanstack/react-query';

// Wire React Query to connectivity status
onlineManager.setEventListener((setOnline) => {
  return NetInfo.addEventListener((state) => {
    setOnline(!!state.isConnected);
  });
});
```

React Query will automatically pause mutations and re-fetch when connectivity is restored.

**Persist cache across app restarts:**

```ts
import AsyncStorage from '@react-native-async-storage/async-storage';
import { createAsyncStoragePersister } from '@tanstack/query-async-storage-persister';
import { PersistQueryClientProvider } from '@tanstack/react-query-persist-client';

const persister = createAsyncStoragePersister({ storage: AsyncStorage });

<PersistQueryClientProvider client={queryClient} persistOptions={{ persister }}>
  <App />
</PersistQueryClientProvider>
```

## Key Points

- Use `fetch` over `axios` — it works in both native and Cloudflare Workers (API routes)
- Always throw typed errors from fetch wrappers so call sites can distinguish network vs HTTP failures
- `expo-secure-store` for tokens; `AsyncStorage` for non-sensitive persisted state
- `EXPO_PUBLIC_*` vars are baked into the bundle — never put secrets there
- Wire `onlineManager` from React Query to NetInfo for automatic offline/online handling

<!--
Source references:
- https://github.com/expo/skills (plugins/expo/skills/native-data-fetching/SKILL.md)
- https://github.com/expo/skills (plugins/expo/skills/native-data-fetching/references/expo-router-loaders.md)
- https://docs.expo.dev/versions/latest/sdk/securestore/
-->
