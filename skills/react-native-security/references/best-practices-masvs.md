---
name: react-native-security-masvs
description: OWASP MASVS v2 control groups mapped to concrete React Native and Expo actions, plus a pre-release security review checklist.
---

# OWASP MASVS Checklist for React Native

OWASP MASVS v2 groups mobile security requirements into eight control groups. The table maps each to what it means in a React Native codebase. Use the OWASP MASTG for test procedures.

| Group | Goal | React Native actions |
|-------|------|----------------------|
| **MASVS-STORAGE** | Sensitive data at rest is protected | Tokens in Keychain/Keystore; encrypted MMKV/SQLCipher for sensitive caches; no PII in AsyncStorage, logs, or crash reports; `allowBackup` reviewed |
| **MASVS-CRYPTO** | Cryptography is used correctly | Use platform crypto (`expo-crypto`, Keychain/Keystore, `crypto.subtle` where available); CSPRNG for keys and nonces; no hard-coded keys; no home-made ciphers; no `Math.random()` for secrets |
| **MASVS-AUTH** | Authentication and authorization are sound | OAuth + PKCE in system browser; server enforces authorization; biometrics bound to Keychain items; step-up for sensitive actions |
| **MASVS-NETWORK** | Data in transit is secure | HTTPS only; no ATS/cleartext exceptions in release; public key pinning for high-value APIs; no user CAs in release |
| **MASVS-PLATFORM** | Platform features are used securely | Validate deep link / universal link params; WebViews: `originWhitelist`, no `javaScriptEnabled` for untrusted content, validate `onMessage` data; minimum permissions; `FLAG_SECURE` on sensitive screens; no sensitive data in clipboard or notifications |
| **MASVS-CODE** | Code and dependencies are maintained and safe | Keep RN/Expo current; `npm audit` / Dependabot / Renovate; lockfile committed; validate all input (zod); no `eval` / dynamic code from network; OTA updates signed (EAS Update code signing) |
| **MASVS-RESILIENCE** | App resists reverse engineering and tampering | R8 on, Hermes bytecode, debug off in release; integrity attestation verified server-side; tamper/hook signals reported |
| **MASVS-PRIVACY** | User privacy is protected | Data minimization; iOS privacy manifest (`PrivacyInfo.xcprivacy`) and required-reason APIs; Play Data safety form; consent before analytics; no device identifiers without need |

## React Native–specific pitfalls

| Pitfall | Fix |
|---------|-----|
| `WebView` loading remote content with `injectedJavaScript` and a permissive `onMessage` | Check `event.nativeEvent.url` origin, parse and validate messages, set `originWhitelist` |
| Deep link `myapp://reset?token=...` handled without validation | Prefer universal/app links; validate params; never auto-execute actions from links |
| OTA updates (EAS Update / CodePush) without signing | Enable EAS Update code signing so only your key can publish runnable JS |
| Sensitive values in Redux/Zustand persisted to plain storage | Exclude them from `persist` or use an encrypted storage adapter |
| Error boundaries / Sentry capturing request bodies | Configure `beforeSend` to scrub tokens and PII |
| Push notification payload containing sensitive data | Send an ID; fetch details after unlock |
| Clipboard holding OTPs or passwords | Avoid programmatic copy; clear after use if required |

## Pre-release review

```
[ ] No secrets in JS, EXPO_PUBLIC_*, app.json extra, or native resources
[ ] Release build: dev menu off, console stripped, source maps private
[ ] Tokens only in Keychain/Keystore; logout wipes all local data
[ ] HTTPS only; pinning configured with backup pins and expiration (if used)
[ ] OAuth with PKCE; redirect URIs are universal/app links
[ ] Deep links, push payloads, WebView messages validated
[ ] Permissions are minimal and justified in usage strings
[ ] Sensitive screens protected from capture
[ ] Integrity tokens verified server-side for sensitive endpoints
[ ] Dependencies audited; RN/Expo on a supported version
[ ] Privacy manifest and Data safety form match actual collection
[ ] EAS Update code signing enabled (if shipping OTA)
```

## Key Points

- MASVS gives you a vocabulary for threat modeling and audits; MASTG gives you tests.
- Most React Native findings are storage, logging, WebView and deep-link issues, not exotic crypto bugs.
- The server enforces the security model; the client protects data at rest, in transit, and on screen.
