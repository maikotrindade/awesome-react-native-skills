# Advanced — Deep Linking

Deep linking opens your app at a specific screen via a URL. Two mechanisms exist: **URL schemes** (custom protocol) and **Universal/App Links** (real HTTPS URLs).

| Type | iOS | Android | Requires server |
|------|-----|---------|----------------|
| URL scheme | `myapp://` | `myapp://` | No |
| Universal Links | `https://example.com/...` | — | Yes (`apple-app-site-association`) |
| App Links | — | `https://example.com/...` | Yes (`assetlinks.json`) |

Prefer Universal Links / App Links for shareable URLs — they fall back gracefully to the website if the app isn't installed.

---

## URL Scheme Setup

### iOS — Info.plist
```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>myapp</string>
    </array>
  </dict>
</array>
```

### Android — AndroidManifest.xml
```xml
<intent-filter>
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="myapp" />
</intent-filter>
```

---

## Universal Links (iOS)

**Step 1 — Apple App Site Association file**

Host at `https://example.com/.well-known/apple-app-site-association` (no extension):
```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "TEAMID.com.example.myapp",
        "paths": ["/post/*", "/profile/*", "/invite/*"]
      }
    ]
  }
}
```

**Step 2 — Xcode Entitlements**

Add Associated Domains capability:
```
applinks:example.com
```

---

## App Links (Android)

**Step 1 — assetlinks.json**

Host at `https://example.com/.well-known/assetlinks.json`:
```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.example.myapp",
    "sha256_cert_fingerprints": ["YOUR:CERT:FINGERPRINT:HERE"]
  }
}]
```

Get your fingerprint:
```bash
keytool -list -v -keystore my-release.keystore -alias my-key-alias
```

**Step 2 — AndroidManifest.xml**
```xml
<intent-filter android:autoVerify="true">
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="https" android:host="example.com" android:pathPrefix="/post" />
</intent-filter>
```

---

## React Navigation Linking Config

```ts
// navigation/linking.ts
import type { LinkingOptions } from '@react-navigation/native';
import type { RootStackParamList } from './types';

export const linking: LinkingOptions<RootStackParamList> = {
  prefixes: [
    'myapp://',
    'https://example.com',
    'https://www.example.com',
  ],
  config: {
    screens: {
      Auth: {
        screens: {
          Login: 'login',
          Register: 'register',
        },
      },
      Main: {
        screens: {
          Home: 'home',
          Profile: 'profile/:userId',      // :userId becomes route param
          Post: {
            path: 'post/:postId',
            parse: { postId: String },     // coerce param type
          },
        },
      },
      Modal: 'modal/:title',
    },
  },
};

// App.tsx
<NavigationContainer linking={linking} fallback={<ActivityIndicator />}>
```

---

## Handling Initial URL (Cold Start)

When the app is launched from a closed state via a link, React Navigation handles it automatically via `getInitialURL`. The `linking` config routes the URL to the correct screen on startup.

For manual handling (e.g., auth-gated deep links):

```ts
import { Linking } from 'react-native';

useEffect(() => {
  Linking.getInitialURL().then((url) => {
    if (url) handleDeepLink(url);
  });
}, []);
```

---

## Warm/Foreground Listening

```ts
useEffect(() => {
  const subscription = Linking.addEventListener('url', ({ url }) => {
    handleDeepLink(url);
  });
  return () => subscription.remove();
}, []);
```

---

## Auth-Gated Deep Links

Store the pending URL and redirect after authentication:

```ts
// store/useLinkingStore.ts
export const useLinkingStore = create<{ pendingUrl: string | null; setPendingUrl: (url: string | null) => void }>((set) => ({
  pendingUrl: null,
  setPendingUrl: (url) => set({ pendingUrl: url }),
}));

// In NavigationContainer
function AppNavigator() {
  const { pendingUrl, setPendingUrl } = useLinkingStore();
  const { user } = useAuthStore();

  useEffect(() => {
    if (user && pendingUrl) {
      // User just logged in — redirect to the pending deep link
      Linking.openURL(pendingUrl);
      setPendingUrl(null);
    }
  }, [user, pendingUrl]);
}

// Custom linking getInitialURL
const linking: LinkingOptions<RootStackParamList> = {
  prefixes: ['myapp://', 'https://example.com'],
  async getInitialURL() {
    const url = await Linking.getInitialURL();
    if (url && !isAuthenticated()) {
      useLinkingStore.getState().setPendingUrl(url);
      return null; // block navigation, handle post-auth
    }
    return url;
  },
  config: { screens: { /* ... */ } },
};
```

---

## Testing Deep Links

```bash
# iOS Simulator
xcrun simctl openurl booted "myapp://profile/42"
xcrun simctl openurl booted "https://example.com/post/99"

# Android Emulator
adb shell am start -W -a android.intent.action.VIEW -d "myapp://profile/42" com.example.myapp
adb shell am start -W -a android.intent.action.VIEW -d "https://example.com/post/99" com.example.myapp
```

---

## Passing Params Through Links

```
myapp://post/42?ref=share&utm_source=twitter
↓
navigation.navigate('Post', { postId: '42', ref: 'share' })
```

Query params are automatically parsed and passed to the screen when defined in `config.screens`:
```ts
Post: {
  path: 'post/:postId',
  parse: {
    postId: String,
    ref: String,       // query param
  },
},
```
