---
name: react-native-security-secrets-bundle
description: Keeping secrets out of React Native apps — what ships in the JS bundle and native binary, EXPO_PUBLIC_ and react-native-config rules, backend proxies, EAS secrets, source maps, Hermes bytecode and R8.
---

# Secrets & the App Bundle

## Rule

**Anything the app can read, an attacker can read.** The JS bundle (Hermes bytecode or plain JS), `Info.plist`, `AndroidManifest.xml`, `strings.xml`, `app.json` `extra`, and every native library can be extracted from the IPA/APK in minutes. Obfuscation slows this down; it does not stop it.

| Value | OK in the app? | Notes |
|-------|----------------|-------|
| Public API base URL | Yes | |
| Publishable keys (Stripe `pk_`, Supabase anon key, Firebase config, Sentry DSN) | Yes | Designed to be public; protect with server rules (RLS, Firebase rules, key restrictions) |
| Maps / Places API key | Yes, if restricted | Restrict by bundle ID / SHA-1 and API scope in the provider console |
| OpenAI/Anthropic/LLM keys, Stripe `sk_`, AWS keys, DB credentials, OAuth client secrets | **No** | Call through your backend |
| Signing keys, keystore passwords | **No** | CI secrets only (EAS secrets, GitHub Actions secrets) |

## Environment variables

**Expo:** only variables prefixed `EXPO_PUBLIC_` are inlined into the bundle — and they are inlined in plain text.

```bash
# .env — committed defaults, public values only
EXPO_PUBLIC_API_URL=https://api.example.com

# Build-time-only secret (used by app.config.ts or config plugins, never by app code)
eas env:create --name SENTRY_AUTH_TOKEN --value ... --visibility secret --environment production
```

```ts
// app code
const apiUrl = process.env.EXPO_PUBLIC_API_URL; // inlined at build time
```

- Non-prefixed variables are available to `app.config.ts`, Expo Router API routes, and EAS Build hooks — not to client code.
- Don't copy a secret into `extra` in `app.config.ts`: `Constants.expoConfig.extra` ships in the app manifest.
- `.env*.local` files belong in `.gitignore`.

**Bare React Native (`react-native-config`, `react-native-dotenv`):** every value you define is compiled into the app. Treat them exactly like `EXPO_PUBLIC_`.

## Backend proxy pattern

```ts
// Client: authenticated call to your server, never to the provider
const res = await fetch(`${process.env.EXPO_PUBLIC_API_URL}/ai/complete`, {
  method: 'POST',
  headers: { Authorization: `Bearer ${accessToken}`, 'Content-Type': 'application/json' },
  body: JSON.stringify({ prompt }),
});
```

```ts
// Server (e.g. app/api/ai/complete+api.ts on EAS Hosting)
export async function POST(req: Request) {
  const user = await verifySession(req);           // authn
  await enforceQuota(user.id);                      // rate limit / abuse control
  const { prompt } = await req.json();
  const upstream = await callProvider(prompt, process.env.PROVIDER_API_KEY!); // secret stays here
  return Response.json(upstream);
}
```

The proxy is also where you attach App Attest / Play Integrity verification (see `features-device-integrity.md`).

## What else leaks

| Leak | Fix |
|------|-----|
| Source maps shipped or hosted publicly | Upload to Sentry/Bugsnag with an auth token; don't include `.map` files in the app or a public CDN |
| `console.log` of tokens, requests, user data | Strip in release with `babel-plugin-transform-remove-console`; use a logger with redaction |
| Dev menu / debugger in release | Ship release builds only (`--configuration Release`, `assembleRelease`, EAS `production` profile) |
| Sensitive strings in native code | Same rule — native binaries are extractable |
| Third-party SDK keys in `google-services.json` / `GoogleService-Info.plist` | Public by design; lock down with Firebase App Check and security rules |
| Git history | Rotate any secret that was ever committed; scan with `gitleaks` or GitHub secret scanning |

## Hardening the binary

- **Hermes bytecode** is the default. It hides source text but can be decompiled; don't rely on it for secrecy.
- **R8 / ProGuard** (Android): on by default for release builds since Expo SDK 58 (`android.enableMinifyInReleaseBuilds`). Bare apps set `minifyEnabled true` in `android/app/build.gradle` and keep rules for reflection-based libraries.
- **JS obfuscators** (e.g. `javascript-obfuscator` via Metro transformers) raise effort, cost bundle size and startup time, and can break Hermes optimizations. Use only if a threat model requires it.
- **iOS**: Release builds strip symbols by default; keep dSYMs private.

## Key Points

- If the app needs it at runtime, it is public. Restrict public keys at the provider; move private keys behind your API.
- `EXPO_PUBLIC_*` and every `react-native-config` value are plain text in the bundle.
- Build-time secrets live in EAS / CI secret stores and are read by config, not by app code.
- Keep source maps and dSYMs private; strip console output in release; rotate anything that was ever committed.
