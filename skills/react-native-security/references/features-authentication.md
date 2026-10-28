---
name: react-native-security-authentication
description: Authentication in React Native — OAuth 2.0 / OIDC with PKCE via expo-auth-session or react-native-app-auth, token storage and refresh, biometric unlock with expo-local-authentication, and session handling.
---

# Authentication

## OAuth 2.0 / OIDC rules for native apps

- **Authorization Code + PKCE** — the only flow to use. Implicit flow and the password grant are out.
- **System browser** (ASWebAuthenticationSession / Custom Tabs) — never a WebView you control; users can't verify the URL and you could read their password.
- **No client secret in the app** — mobile apps are public clients.
- **Claimed redirect URIs** — prefer universal links / app links (`https://app.example.com/oauth`) over custom schemes, which any app can register.
- **Validate `state`** (the libraries below do this) and, for OIDC, verify the ID token on the server.

## Expo: expo-auth-session

```bash
npx expo install expo-auth-session expo-crypto expo-web-browser
```

```tsx
import * as AuthSession from 'expo-auth-session';
import * as WebBrowser from 'expo-web-browser';

WebBrowser.maybeCompleteAuthSession(); // required for web redirects

export function SignInButton() {
  const discovery = AuthSession.useAutoDiscovery('https://auth.example.com');
  const redirectUri = AuthSession.makeRedirectUri({ scheme: 'myapp', path: 'oauth' });

  const [request, response, promptAsync] = AuthSession.useAuthRequest(
    {
      clientId: 'mobile-app',
      scopes: ['openid', 'profile', 'offline_access'],
      redirectUri,
      usePKCE: true, // default — keep it on
    },
    discovery,
  );

  useEffect(() => {
    if (response?.type !== 'success' || !discovery || !request?.codeVerifier) return;
    AuthSession.exchangeCodeAsync(
      {
        clientId: 'mobile-app',
        code: response.params.code,
        redirectUri,
        extraParams: { code_verifier: request.codeVerifier },
      },
      discovery,
    ).then(saveTokens);
  }, [response]);

  return <Button title="Sign in" disabled={!request} onPress={() => promptAsync()} />;
}
```

Exchanging the code on your own backend (send `code` + `code_verifier` to it) is preferable when the backend issues its own session.

## Bare React Native: react-native-app-auth

```ts
import { authorize, refresh, revoke } from 'react-native-app-auth';

const config = {
  issuer: 'https://auth.example.com',
  clientId: 'mobile-app',
  redirectUrl: 'com.example.app:/oauth',
  scopes: ['openid', 'profile', 'offline_access'],
  // PKCE is on by default
};

const result = await authorize(config);
await saveTokens(result); // accessToken, refreshToken, idToken, accessTokenExpirationDate

const renewed = await refresh(config, { refreshToken });
await revoke(config, { tokenToRevoke: refreshToken, sendClientId: true });
```

Register the redirect scheme in `Info.plist` and `build.gradle` (`manifestPlaceholders = [appAuthRedirectScheme: 'com.example.app']`).

## Token lifecycle

| Token | Lifetime | Storage |
|-------|----------|---------|
| Access token | Minutes | Memory (state/store); Keychain only if background work needs it |
| Refresh token | Days–weeks, rotated on use | Keychain / SecureStore, `*_THIS_DEVICE_ONLY` |
| ID token | Single use at sign-in | Don't persist; read claims, then drop |

```ts
// Single-flight refresh so parallel 401s don't race
let refreshing: Promise<string> | null = null;

export function getFreshAccessToken() {
  refreshing ??= refreshTokens().finally(() => { refreshing = null; });
  return refreshing;
}
```

On refresh failure (revoked, expired, reuse detected): clear secure storage and caches, then route to sign-in.

## Biometric unlock

Biometrics prove presence on *this device*; they don't replace server authentication. Gate access to a stored refresh token, not a boolean flag:

```ts
import * as LocalAuthentication from 'expo-local-authentication';

const canUse =
  (await LocalAuthentication.hasHardwareAsync()) &&
  (await LocalAuthentication.isEnrolledAsync());

if (canUse) {
  const { success } = await LocalAuthentication.authenticateAsync({
    promptMessage: 'Unlock',
    disableDeviceFallback: false,
  });
  if (success) unlockSession();
}
```

`authenticateAsync` returning `success` is a JS boolean an attacker with a hooked runtime can flip. For real binding, store the refresh token with `requireAuthentication: true` (SecureStore) or `accessControl: BIOMETRY_CURRENT_SET` (Keychain) so the OS releases the secret only after a successful biometric check. Add `NSFaceIDUsageDescription` on iOS.

## Session hygiene

- Re-authenticate (or step-up with biometrics) before sensitive actions: changing email/password, payments, exporting data.
- Hide sensitive screens in the app switcher (see `features-device-integrity.md`).
- Sign-out revokes the refresh token server-side, then wipes local storage.

## Key Points

- Authorization Code + PKCE in the system browser; no client secrets, no WebView logins.
- Access token in memory, rotating refresh token in secure storage, single-flight refresh.
- Biometric unlock should release a Keychain-bound secret, not just flip a flag.
