---
name: react-native-security-device-integrity
description: Device and app integrity for React Native — jailbreak/root signals with jail-monkey, server-verified App Attest and Play Integrity via @expo/app-integrity, RASP options, and screen-capture / app-switcher protection.
---

# Device & App Integrity

## Two kinds of checks

| | Local checks | Platform attestation |
|---|---|---|
| Examples | Jailbreak/root files, hooking frameworks, debugger, emulator | Apple App Attest, Google Play Integrity |
| Runs | On device, result in JS | On device, **verified on your server** |
| Bypass | Easy (Frida, Magisk/Zygisk hide, patched binary) | Hard — signed by Apple/Google |
| Use for | UX warnings, telemetry, raising cost | Gating sensitive API calls |

Never make a security decision from a boolean computed in JS. Send attestation tokens to the backend and decide there.

## Local signals: jail-monkey

```bash
npm install jail-monkey   # native module — dev build on Expo
```

```ts
import JailMonkey from 'jail-monkey';

export async function deviceRiskSignals() {
  return {
    jailbroken: JailMonkey.isJailBroken(),
    hooked: JailMonkey.hookDetected(),
    debugged: await JailMonkey.isDebuggedMode(),
    mockLocation: JailMonkey.canMockLocation(),        // Android
    externalStorage: JailMonkey.isOnExternalStorage(), // Android
    adb: JailMonkey.AdbEnabled(),                      // Android
  };
}
```

Report signals with your telemetry and show a warning; blocking outright frustrates legitimate users on custom ROMs and stops no determined attacker.

For stronger runtime protection (anti-tampering, anti-hooking, repackaging detection with callbacks), RASP SDKs such as `freerasp-react-native` run checks natively and can report to a dashboard. They raise the bar but are still bypassable locally.

## Platform attestation: @expo/app-integrity

Works in Expo and bare apps (Expo Modules). Needs a dev build.

```bash
npx expo install @expo/app-integrity
```

**iOS — App Attest**

```ts
import * as AppIntegrity from '@expo/app-integrity';

// Once per install: create and attest a key
const keyId = await AppIntegrity.generateKeyAsync();
const challenge = await api.getChallenge();                    // server nonce
const attestation = await AppIntegrity.attestKeyAsync(keyId, challenge);
await api.registerAttestedKey({ keyId, attestation });         // server verifies with Apple

// Per sensitive request: sign the request
const reqChallenge = await api.getChallenge();
const assertion = await AppIntegrity.generateAssertionAsync(keyId, reqChallenge);
await api.transfer(payload, { keyId, assertion });             // server verifies assertion + counter
```

Store `keyId` in secure storage. Check `AppIntegrity.isSupported` first — App Attest is unavailable on simulators and some older devices. Enable the App Attest capability and set the environment to `production` for release builds.

**Android — Play Integrity**

```ts
// Warm up once (e.g. app start)
await AppIntegrity.prepareIntegrityTokenProviderAsync(GOOGLE_CLOUD_PROJECT_NUMBER);

// Per request: bind the token to the request contents
const requestHash = sha256(JSON.stringify(payload)); // e.g. via expo-crypto digestStringAsync
const token = await AppIntegrity.requestIntegrityCheckAsync(requestHash);
await api.transfer(payload, { integrityToken: token });
```

The server decrypts the token with Google's API and checks `requestHash`, package name, certificate digest, `appRecognitionVerdict` (`PLAY_RECOGNIZED`) and `deviceRecognitionVerdict` (`MEETS_DEVICE_INTEGRITY` or stronger). Android also exposes Key Attestation (`generateHardwareAttestedKeyAsync`, `getAttestationCertificateChainAsync`) for hardware-backed key proof.

**Firebase alternative:** `@react-native-firebase/app-check` wraps App Attest / Play Integrity and enforces them for Firebase services and your own backend via App Check tokens.

### Server-side policy

- Issue fresh, single-use challenges from the server; reject replays.
- Decide per endpoint: block, require step-up auth, or allow and flag.
- Degrade gracefully when attestation is unsupported (old devices, sideloaded test builds) — log it, don't brick.
- Watch quotas: Play Integrity has daily limits; cache verdicts for low-risk calls.

## Screen capture & app switcher

```bash
npx expo install expo-screen-capture
```

```tsx
import * as ScreenCapture from 'expo-screen-capture';

function CardDetailsScreen() {
  ScreenCapture.usePreventScreenCapture(); // Android: FLAG_SECURE; iOS: blanks screen recordings
  ScreenCapture.useScreenshotListener(() => analytics.track('screenshot_on_sensitive_screen'));
  return <CardDetails />;
}

// Blur the app switcher snapshot (iOS)
await ScreenCapture.enableAppSwitcherProtectionAsync();
```

Use it on screens with credentials, card numbers, OTPs, or private messages — not app-wide, since it also blocks legitimate screenshots and screen sharing.

## Key Points

- Local jailbreak/root/hook checks are signals for telemetry and UX, not access control.
- App Attest and Play Integrity tokens are only meaningful when verified on the server with a fresh challenge.
- Bind attestation to the request (`requestHash` / assertion over the challenge) for sensitive operations.
- Protect sensitive screens from capture and the app switcher, scoped to those screens.
