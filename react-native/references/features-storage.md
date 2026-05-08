# Features — Storage

## Decision Matrix

| Library | Speed | API | Use case |
|---------|-------|-----|---------|
| **MMKV** | Fastest (C++) | Synchronous | App state, settings, flags, frequent reads |
| **AsyncStorage** | Medium | Async | Simple key-value persistence, migration compat |
| **Expo SecureStore** | Medium | Async | Credentials, tokens, sensitive data |
| **SQLite / Drizzle** | Fast for queries | Async | Relational data, offline-first, complex queries |

---

## MMKV

10× faster than AsyncStorage. Synchronous API means no async/await overhead.

### Installation
```bash
npm install react-native-mmkv
```

### Basic Usage
```ts
import { MMKV } from 'react-native-mmkv';

export const storage = new MMKV();

// Write
storage.set('username', 'maiko');
storage.set('age', 30);
storage.set('isOnboarded', true);

// Read (synchronous — no await needed)
const username = storage.getString('username');   // string | undefined
const age = storage.getNumber('age');             // number | undefined
const onboarded = storage.getBoolean('isOnboarded'); // boolean | undefined

// Delete
storage.delete('username');

// Check existence
const exists = storage.contains('username');

// All keys
const allKeys = storage.getAllKeys();

// Clear everything
storage.clearAll();
```

### Multiple Instances (namespacing)
```ts
// Separate storage instances for isolation
export const userStorage = new MMKV({ id: 'user-storage' });
export const cacheStorage = new MMKV({ id: 'cache-storage' });
export const sessionStorage = new MMKV({ id: 'session-storage' });
```

### Encryption
```ts
// Encrypted MMKV instance — key must be stored securely (e.g., in Keychain)
export const secureStorage = new MMKV({
  id: 'secure-storage',
  encryptionKey: 'my-secret-key', // fetch from native keychain in production
});
```

### Zustand Adapter
```ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { storage } from './storage';

const zustandMMKVStorage = createJSONStorage(() => ({
  getItem: (key: string) => storage.getString(key) ?? null,
  setItem: (key: string, value: string) => storage.set(key, value),
  removeItem: (key: string) => storage.delete(key),
}));

export const useSettingsStore = create(
  persist(
    (set) => ({
      theme: 'light' as 'light' | 'dark',
      setTheme: (theme: 'light' | 'dark') => set({ theme }),
    }),
    { name: 'settings', storage: zustandMMKVStorage }
  )
);
```

### Listen for Changes
```ts
// React to storage changes (e.g., from a different thread)
const listener = storage.addOnValueChangedListener((changedKey) => {
  if (changedKey === 'theme') {
    applyTheme(storage.getString('theme'));
  }
});

// Remove listener on cleanup
listener.remove();
```

---

## AsyncStorage

```bash
npm install @react-native-async-storage/async-storage
```

```ts
import AsyncStorage from '@react-native-async-storage/async-storage';

// Store string
await AsyncStorage.setItem('token', authToken);

// Store object (must serialize)
await AsyncStorage.setItem('user', JSON.stringify(user));

// Read
const token = await AsyncStorage.getItem('token');
const user = JSON.parse((await AsyncStorage.getItem('user')) ?? 'null');

// Delete
await AsyncStorage.removeItem('token');

// Batch operations (more efficient than multiple setItem calls)
await AsyncStorage.multiSet([
  ['key1', 'value1'],
  ['key2', 'value2'],
]);

const values = await AsyncStorage.multiGet(['key1', 'key2']);
// values: [['key1', 'value1'], ['key2', 'value2']]

// Clear all
await AsyncStorage.clear();

// All keys
const keys = await AsyncStorage.getAllKeys();
```

---

## Expo SecureStore

Backed by iOS Keychain and Android Keystore. Items are tied to the device; size limit is ~2KB per item.

```bash
npx expo install expo-secure-store
```

```ts
import * as SecureStore from 'expo-secure-store';

// Store
await SecureStore.setItemAsync('authToken', token);

// Retrieve
const token = await SecureStore.getItemAsync('authToken');

// Delete
await SecureStore.deleteItemAsync('authToken');

// With biometric authentication requirement
await SecureStore.setItemAsync('biometricToken', token, {
  requireAuthentication: true,
  authenticationPrompt: 'Authenticate to access your account',
});
```

---

## SQLite with Drizzle ORM

For relational data, offline-first apps, or when you need queries/joins.

```bash
npx expo install expo-sqlite
npm install drizzle-orm
npm install -D drizzle-kit
```

```ts
// db/schema.ts
import { sqliteTable, text, integer } from 'drizzle-orm/sqlite-core';

export const users = sqliteTable('users', {
  id: text('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  createdAt: integer('created_at', { mode: 'timestamp' }).notNull(),
});

// db/client.ts
import { drizzle } from 'drizzle-orm/expo-sqlite';
import { openDatabaseSync } from 'expo-sqlite';

const expo = openDatabaseSync('app.db', { enableChangeListener: true });
export const db = drizzle(expo);

// Queries
const allUsers = await db.select().from(users);
const user = await db.select().from(users).where(eq(users.id, '1')).get();

await db.insert(users).values({ id: '1', name: 'Maiko', email: 'a@b.com', createdAt: new Date() });
await db.update(users).set({ name: 'Updated' }).where(eq(users.id, '1'));
await db.delete(users).where(eq(users.id, '1'));
```

---

## Migration Strategy

When migrating from AsyncStorage to MMKV:

```ts
async function migrateToMMKV() {
  const migrated = storage.getBoolean('mmkv_migrated');
  if (migrated) return;

  const keys = await AsyncStorage.getAllKeys();
  const pairs = await AsyncStorage.multiGet(keys);

  for (const [key, value] of pairs) {
    if (value !== null) storage.set(key, value);
  }

  storage.set('mmkv_migrated', true);
  await AsyncStorage.clear();
}
```
