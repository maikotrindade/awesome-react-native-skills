---
name: react-native-security
description: React Native app security — Keychain/Keystore storage, encrypted MMKV, certificate pinning, jailbreak/root and app integrity checks, OAuth with PKCE, secrets kept out of the JS bundle, and an OWASP MASVS checklist.
metadata:
  author: maikotrindade
  version: "2026.10.0"
---

# React Native Security

> Covers client-side hardening for React Native 0.8x and Expo SDK 54–58. A mobile client is never trusted: everything here raises the cost of an attack, and the server still has to enforce authorization, rate limits and integrity verdicts. For generic storage choices see `react-native-ecosystem/features-storage.md`.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Secure storage | expo-secure-store, react-native-keychain v10, biometric-gated secrets, encrypted MMKV, what never to store | [core-secure-storage](references/core-secure-storage.md) |
| Secrets & bundle | What ends up in the JS bundle, `EXPO_PUBLIC_` rules, backend proxies, source maps, Hermes bytecode, R8 | [core-secrets-bundle](references/core-secrets-bundle.md) |

## Features

| Topic | Description | Reference |
|-------|-------------|-----------|
| Network security | HTTPS only, ATS and Android network security config, public key pinning, pin rotation | [features-network-security](references/features-network-security.md) |
| Authentication | OAuth 2.0 + PKCE with expo-auth-session / react-native-app-auth, token lifecycle, biometric unlock | [features-authentication](references/features-authentication.md) |
| Device & app integrity | Jailbreak/root signals, Play Integrity, App Attest, RASP, screenshot protection | [features-device-integrity](references/features-device-integrity.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| MASVS checklist | OWASP MASVS v2 control groups mapped to React Native actions, release review | [best-practices-masvs](references/best-practices-masvs.md) |

## Key Recommendations

- **Tokens go in the Keychain/Keystore** — `expo-secure-store` or `react-native-keychain`, never AsyncStorage or plain MMKV
- **No secrets in the bundle** — anything in JS, `EXPO_PUBLIC_*`, or `app.json` `extra` ships to every user; call third-party APIs through your backend
- **Pin public keys, not certificates** — at least two pins per domain (one backup), with an expiration date so old builds don't brick
- **PKCE for every OAuth flow** — system browser only, no embedded WebView login, no client secret in the app
- **Integrity checks are signals, not gates** — send Play Integrity / App Attest tokens to the server and decide there; local jailbreak checks are bypassable
- **Strip debug surface in release** — no `console.log` of tokens, no source maps uploaded publicly, R8 on for Android release builds (default since Expo SDK 58)
