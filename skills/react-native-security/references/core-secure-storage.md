---
name: react-native-security-secure-storage
description: Storing secrets on device — expo-secure-store, react-native-keychain v10, biometric-gated items, encrypted MMKV v4 with a Keychain-held key, and data that must never be persisted.
---

# Secure Storage

## Where data belongs

| Data | Store | Why |
|------|-------|-----|
| Access/refresh tokens, session IDs | Keychain (iOS) / Keystore-backed storage (Android) | Hardware-backed encryption, excluded from backups when `*_THIS_DEVICE_ONLY` |
| Encryption keys for local databases | Keychain / Keystore | Key never sits next to the data it protects |
| Cached user data that is sensitive (PII, health, finance) | Encrypted MMKV or SQLCipher, key in Keychain | Fast reads, ciphertext at rest |
| Preferences, feature flags, UI state | Plain MMKV / AsyncStorage | Not sensitive |
| Passwords, card numbers, private keys from a server | **Nowhere** — exchange for a token | Reduce what can leak |

AsyncStorage and unencrypted MMKV write plain files inside the app sandbox. On a rooted/jailbroken device, or through an unencrypted backup, those files are readable.

## Expo: expo-secure-store

```bash
npx expo install expo-secure-store
```

```ts
import * as SecureStore from 'expo-secure-store';

const opts: SecureStore.SecureStoreOptions = {
  keychainAccessible: SecureStore.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
};

await SecureStore.setItemAsync('refreshToken', token, opts);
const token = await SecureStore.getItemAsync('refreshToken', opts);
await SecureStore.deleteItemAsync('refreshToken', opts);
```

- Pass the **same options** (`keychainService`, `keychainAccessible`) on read, write and delete — items are looked up by them.
- `requireAuthentication: true` gates the item behind biometrics/passcode. On iOS the prompt appears on read/update; on Android every operation needs authentication. Add the config plugin with `faceIDPermission` for the iOS usage string, and test on a real device (simulators don't enforce it).
- Biometric enrollment changes invalidate authenticated items — handle the read error by signing the user in again.
- Values are strings. Keep them small (tokens, keys); store large blobs encrypted elsewhere.
- Sync `getItem`/`setItem` exist but block the JS thread on a Keychain call; prefer the async versions outside of startup.

## Bare React Native: react-native-keychain v10

```bash
npm install react-native-keychain
cd ios && pod install
```

```ts
import * as Keychain from 'react-native-keychain';

await Keychain.setGenericPassword('session', refreshToken, {
  service: 'com.example.auth',
  accessible: Keychain.ACCESSIBLE.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
  securityLevel: Keychain.SECURITY_LEVEL.SECURE_HARDWARE, // Android: refuse software-only Keystore
});

const creds = await Keychain.getGenericPassword({ service: 'com.example.auth' });
if (creds) use(creds.password);

await Keychain.resetGenericPassword({ service: 'com.example.auth' });
```

**Biometric-gated item:**

```ts
await Keychain.setGenericPassword('session', refreshToken, {
  service: 'com.example.auth.bio',
  accessControl: Keychain.ACCESS_CONTROL.BIOMETRY_CURRENT_SET_OR_DEVICE_PASSCODE,
  accessible: Keychain.ACCESSIBLE.WHEN_PASSCODE_SET_THIS_DEVICE_ONLY,
});

const creds = await Keychain.getGenericPassword({
  service: 'com.example.auth.bio',
  authenticationPrompt: { title: 'Unlock your account' },
});
```

| Option | Use |
|--------|-----|
| `ACCESSIBLE.WHEN_UNLOCKED_THIS_DEVICE_ONLY` | Default choice for tokens: unreadable while locked, not migrated to new devices |
| `ACCESSIBLE.AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY` | Background refresh (push handlers, background fetch) needs the token |
| `ACCESS_CONTROL.BIOMETRY_CURRENT_SET` | Invalidate when fingerprints/faces change — strongest binding |
| `ACCESS_CONTROL.BIOMETRY_ANY_OR_DEVICE_PASSCODE` | Survives enrollment changes, falls back to passcode |
| `SECURITY_LEVEL.SECURE_HARDWARE` | Android: require TEE/StrongBox-backed keys |
| `STORAGE_TYPE.AES_GCM` / `AES_GCM_NO_AUTH` | Android cipher choice; leave unset to get the best available |
| `cloudSync: false` (default) | Keep items out of iCloud Keychain |

Use `getSupportedBiometryType()` and `getSecurityLevel()` to decide what to offer the user before storing.

## Encrypted MMKV v4

MMKV v4 is a Nitro module created with `createMMKV`. Keep its key in the Keychain, never in source:

```ts
import { createMMKV } from 'react-native-mmkv';
import * as SecureStore from 'expo-secure-store';
import * as Crypto from 'expo-crypto';

const KEY_NAME = 'mmkv-key';

async function getOrCreateKey() {
  let key = await SecureStore.getItemAsync(KEY_NAME);
  if (!key) {
    const bytes = Crypto.getRandomBytes(16);
    key = Array.from(bytes, (b) => b.toString(16).padStart(2, '0')).join(''); // 32 chars
    await SecureStore.setItemAsync(KEY_NAME, key, {
      keychainAccessible: SecureStore.AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY,
    });
  }
  return key;
}

export async function openSecureStorage() {
  return createMMKV({
    id: 'secure',
    encryptionKey: await getOrCreateKey(),
    encryptionType: 'AES-256', // key up to 32 bytes; AES-128 (default) takes up to 16
  });
}
```

- Rotate or add encryption to an existing instance with `storage.encrypt(newKey, 'AES-256')`; `decrypt()` removes it. `recrypt()` is deprecated.
- Open the encrypted instance after the key resolves — don't create it at module scope with a placeholder key.

## Hygiene

- **Logout clears everything**: Keychain items, encrypted stores, React Query caches, and in-memory state.
- **Reinstall on iOS**: Keychain items survive app deletion. On first launch after install, check a non-Keychain flag (e.g. in MMKV) and wipe stale Keychain items if it's missing.
- **Backups**: set `android:allowBackup="false"` (or exclude sensitive files with `dataExtractionRules`) when you keep sensitive files outside the Keystore.
- **Never log** tokens or Keychain results — release builds still keep `console.*` output in device logs unless you strip it (`babel-plugin-transform-remove-console`).

## Key Points

- Tokens and keys live in the Keychain/Keystore; everything else sensitive is encrypted with a key that lives there.
- Use `*_THIS_DEVICE_ONLY` accessibility unless you have a reason to sync credentials across devices.
- Biometric gating binds a secret to the user's presence; handle invalidation after enrollment changes.
- MMKV v4: `createMMKV({ encryptionKey, encryptionType })`, key generated with a CSPRNG and held in secure storage.
