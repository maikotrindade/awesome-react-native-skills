---
name: react-native-security-network
description: Network security for React Native — HTTPS-only transport, iOS ATS and Android network security config, public key pinning with react-native-ssl-public-key-pinning, pin rotation and failure reporting.
---

# Network Security

## HTTPS only

**iOS (ATS):** App Transport Security blocks plain HTTP by default. Don't add `NSAllowsArbitraryLoads`. If a dev server needs HTTP, scope an exception to that host in the Debug configuration only.

**Android:** cleartext is blocked by default from API 28. Make it explicit and keep debug exceptions out of release:

```xml
<!-- android/app/src/main/res/xml/network_security_config.xml -->
<network-security-config>
  <base-config cleartextTrafficPermitted="false">
    <trust-anchors>
      <certificates src="system" />
    </trust-anchors>
  </base-config>
</network-security-config>
```

```xml
<!-- AndroidManifest.xml -->
<application android:networkSecurityConfig="@xml/network_security_config" ...>
```

Never ship `<certificates src="user" />` in release — it trusts user-installed CAs (the standard MITM proxy setup). In Expo, write these files with a config plugin rather than editing generated native folders.

## Public key pinning

Pinning makes the app reject TLS connections whose certificate chain doesn't contain a key you expect, defeating rogue CAs and proxy certificates.

```bash
npm install react-native-ssl-public-key-pinning   # Expo: needs a dev build, not Expo Go
```

```ts
import {
  initializeSslPinning,
  addSslPinningErrorListener,
  isSslPinningAvailable,
} from 'react-native-ssl-public-key-pinning';

export async function setupPinning() {
  if (!isSslPinningAvailable()) return; // e.g. Expo Go

  await initializeSslPinning({
    'api.example.com': {
      includeSubdomains: true,
      publicKeyHashes: [
        'CURRENT_LEAF_OR_INTERMEDIATE_SPKI_SHA256_BASE64=',
        'BACKUP_KEY_SPKI_SHA256_BASE64=',
      ],
      expirationDate: '2027-06-30', // pinning switches off after this date
    },
  });
}

const sub = addSslPinningErrorListener((e) => {
  report('pinning_failure', { host: e.serverHostname }); // send to your telemetry
});
```

- Call `initializeSslPinning` before the first request (top of the root layout or `index.js`). It applies to `fetch`, Axios and anything built on the platform networking stack.
- iOS requires **at least two pins per domain**. Always include a backup key you hold offline.
- WebViews, and native SDKs with their own HTTP clients, are not covered.

**Getting a pin (SPKI SHA-256, base64):**

```bash
openssl s_client -connect api.example.com:443 -servername api.example.com </dev/null 2>/dev/null \
  | openssl x509 -pubkey -noout \
  | openssl pkey -pubin -outform der \
  | openssl dgst -sha256 -binary | base64
```

### Which key to pin

| Pin | Rotation risk | Protection |
|-----|---------------|------------|
| Leaf key | High — changes on every key rotation | Strongest |
| Intermediate CA key | Medium — changes when the CA rotates | Good, common choice |
| Root CA key | Low | Weakest (any cert from that CA passes) |

Most teams pin the intermediate plus a backup from a second CA, or pin a leaf key that is reused across certificate renewals.

### Rotation plan

1. Ship the next key as a backup pin **before** rotating the server.
2. Set `expirationDate` so builds that never update fail open instead of breaking.
3. Watch the error listener after each server cert change.
4. Keep a remote kill switch (feature flag) that skips `initializeSslPinning` on the next launch if pins go wrong.

## Request hygiene

- Send tokens in the `Authorization` header, never in URLs or query strings (they end up in logs and analytics).
- Short-lived access tokens; refresh with a rotating refresh token held in secure storage.
- Validate and type every response (e.g. zod) — a compromised or misconfigured backend shouldn't crash or exploit the client.
- Deep links and push payloads are untrusted input: validate parameters before navigating or calling APIs.

## Key Points

- HTTPS everywhere; no `NSAllowsArbitraryLoads`, no `cleartextTrafficPermitted="true"`, no user CAs in release.
- Pin SPKI hashes with a backup and an expiration date; report failures; keep a remote off switch.
- Pinning doesn't cover WebViews or SDKs with their own network stack.
