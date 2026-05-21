---
name: react-native-network
description: Network requests in React Native. Covers the built-in Fetch API and when to upgrade to Axios + TanStack Query for production apps.
---

# Networking

React Native includes the standard Fetch API. For production apps, the standard stack is **Axios** (HTTP client) + **TanStack Query v5** (server state management) — covered in the `react-native-ecosystem` skill.

## When to Use What

| Approach | Use case |
|----------|---------|
| `fetch` built-in | Simple scripts, quick prototypes, no external dependencies |
| Axios | Single interceptor layer, better error handling, multipart uploads |
| Axios + TanStack Query | Production apps — caching, background refresh, pagination, optimistic updates |

## Fetch API

### GET Request

```tsx
const getData = async () => {
  const response = await fetch('https://api.example.com/users');

  if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`);
  }

  return response.json();
};
```

### POST Request

```tsx
const postData = async (userData: { name: string; email: string }) => {
  const response = await fetch('https://api.example.com/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: 'Bearer token',
    },
    body: JSON.stringify(userData),
  });

  if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
  return response.json();
};
```

### Request Options

```tsx
fetch(url, {
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE',
  headers: Record<string, string>,
  body: string | FormData | URLSearchParams,
  signal: AbortController.signal,  // for cancellation
});
```

### Abort / Cancel

```tsx
const controller = new AbortController();

useEffect(() => {
  fetchData(controller.signal);
  return () => controller.abort(); // cancel on unmount
}, []);

const fetchData = async (signal: AbortSignal) => {
  const res = await fetch('https://api.example.com/data', { signal });
  return res.json();
};
```

### FormData (file upload)

```tsx
const upload = async (uri: string, name: string) => {
  const formData = new FormData();
  formData.append('file', {
    uri,
    name,
    type: 'image/jpeg',
  } as unknown as Blob);
  formData.append('caption', 'My photo');

  const response = await fetch('https://api.example.com/upload', {
    method: 'POST',
    body: formData,
    headers: { 'Content-Type': 'multipart/form-data' },
  });

  return response.json();
};
```

## Using Fetch in React Components

```tsx
import { useState, useEffect } from 'react';

function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let cancelled = false;

    (async () => {
      try {
        const res = await fetch('https://api.example.com/users');
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const data = await res.json();
        if (!cancelled) setUsers(data);
      } catch (err) {
        if (!cancelled) setError((err as Error).message);
      } finally {
        if (!cancelled) setLoading(false);
      }
    })();

    return () => { cancelled = true; };
  }, []);

  if (loading) return <ActivityIndicator />;
  if (error) return <Text>Error: {error}</Text>;
  return <FlatList data={users} keyExtractor={(u) => u.id} renderItem={({ item }) => <Text>{item.name}</Text>} />;
}
```

> In production, replace the above with `useQuery` from TanStack Query — it handles loading states, caching, deduplication, background refresh, and retries automatically.

## WebSocket

```tsx
const ws = useRef<WebSocket | null>(null);

useEffect(() => {
  ws.current = new WebSocket('wss://api.example.com/ws');

  ws.current.onopen = () => console.log('Connected');
  ws.current.onmessage = (e) => console.log('Message:', e.data);
  ws.current.onerror = (e) => console.log('Error:', e.message);
  ws.current.onclose = (e) => console.log('Closed:', e.code, e.reason);

  return () => ws.current?.close();
}, []);

const sendMessage = (msg: string) => {
  ws.current?.send(JSON.stringify({ type: 'message', data: msg }));
};
```

## Security: Platform Network Config

By default, iOS blocks plain HTTP. Add exceptions only when necessary:

**iOS** (`Info.plist`):
```xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <false/>
  <key>NSExceptionDomains</key>
  <dict>
    <key>localhost</key>
    <dict>
      <key>NSExceptionAllowsInsecureHTTPLoads</key>
      <true/>
    </dict>
  </dict>
</dict>
```

**Android** (`android/app/src/main/res/xml/network_security_config.xml`):
```xml
<network-security-config>
  <domain-config cleartextTrafficPermitted="true">
    <domain includeSubdomains="true">localhost</domain>
  </domain-config>
</network-security-config>
```
