---
name: react-native-performance-tti
description: Measuring Time to Interactive in React Native — cold start detection, startup pipeline markers, and react-native-performance instrumentation.
---

# Time to Interactive (TTI)

TTI = time from app icon tap to displaying usable content. Only **cold starts** give meaningful metrics; warm, hot, and iOS prewarmed starts should be filtered out.

## Startup Types

| Type | Description | Measure? |
|------|-------------|----------|
| Cold | App not in memory, full init | Yes |
| Warm | Process exists, activity recreated | No |
| Hot | App in background, resumed | No |
| Prewarmed (iOS) | iOS pre-initialized app | Filter out |

## React Native Startup Pipeline

```
1. Native Process Init     nativeLaunchStart  → nativeLaunchEnd
2. Native App Init         appCreationStart   → appCreationEnd
3. JS Bundle Load          runJSBundleStart   → runJSBundleEnd
4. RN Root View Render     contentAppeared
5. React App Interactive   screenInteractive   ← TTI endpoint
```

The JS bundle parse/eval is typically the largest cold-start contributor on Hermes. See `react-native-performance/references/best-practices-bundle.md` for bundle-size reduction.

## Setup

```bash
npm install react-native-performance
```

`react-native-performance` ships these markers automatically: `nativeLaunchStart`, `nativeLaunchEnd`, `runJSBundleStart`, `runJSBundleEnd`, `contentAppeared`.

## Detecting Cold Start

**iOS (Swift):**

```swift
let isColdStart = ProcessInfo.processInfo.environment["ActivePrewarm"] != "1"
```

**Android (Kotlin):**

```kotlin
class MainApplication : Application() {
  var isColdStart = false
  override fun onCreate() {
    super.onCreate()
    var firstPostEnqueued = true
    Handler().post { firstPostEnqueued = false }
    registerActivityLifecycleCallbacks(object : ActivityLifecycleCallbacks {
      override fun onActivityCreated(activity: Activity, savedInstanceState: Bundle?) {
        unregisterActivityLifecycleCallbacks(this)
        if (firstPostEnqueued && savedInstanceState == null) isColdStart = true
      }
      // ... other lifecycle callbacks no-op
    })
  }
}
```

## Checking Foreground

Only measure when app starts in foreground (push notifications can launch in background).

**iOS:**

```swift
if application.applicationState == .active { isForegroundProcess = true }
```

**Android:**

```kotlin
val info = ActivityManager.RunningAppProcessInfo()
ActivityManager.getMyMemoryState(info)
return info.importance == IMPORTANCE_FOREGROUND
```

## Native Markers

**iOS (Swift):**

```swift
import ReactNativePerformance

RNPerformance.sharedInstance().mark("appCreationStart")
// ... app init ...
RNPerformance.sharedInstance().mark("appCreationEnd")
```

**Android (Kotlin):**

```kotlin
import com.oblador.performance.RNPerformance

RNPerformance.getInstance().mark("appCreationStart")
// ... app init ...
RNPerformance.getInstance().mark("appCreationEnd")
```

## JS Marker — Screen Interactive

Mark when meaningful content is *interactive*, not just mounted.

```tsx
import performance from 'react-native-performance';

export default function HomeScreen() {
  useEffect(() => {
    performance.mark('screenInteractive');
  }, []);
  return <TabNavigator />;
}
```

## Collecting Metrics

```tsx
import performance from 'react-native-performance';

const collectTTIMetrics = () => {
  const metrics = {
    nativeInit:   getMarkDuration('nativeLaunchStart', 'nativeLaunchEnd'),
    appCreation:  getMarkDuration('appCreationStart', 'appCreationEnd'),
    jsBundleLoad: getMarkDuration('runJSBundleStart', 'runJSBundleEnd'),
    tti:          getMarkDuration('nativeLaunchStart', 'screenInteractive'),
  };
  analytics.track('app_performance', metrics);
};
```

## Listening to Native Events

**iOS** — observe `RCTJavaScriptDidLoadNotification` via `NotificationCenter`.

**Android** — `ReactMarker.addListener { name -> when (name) { RUN_JS_BUNDLE_START, RUN_JS_BUNDLE_END, CONTENT_APPEARED -> ... } }`.

## Target Metrics

| Metric | Good | Acceptable | Needs Work |
|--------|------|------------|------------|
| TTI | < 2s | 2–4s | > 4s |
| JS Bundle Load | < 500ms | 500ms–1s | > 1s |
| Native Init | < 500ms | 500ms–1s | > 1s |

Targets vary by app complexity and device tier.

## Reducing TTI

- **JS bundle size** — biggest lever; tree-shake, code-split, remove polyfills (see `features-native-performance.md`).
- **Hermes mmap** — Android can mmap the bytecode bundle to skip parse cost on cold start.
- **Lazy native modules** — defer non-critical Turbo Module init.
- **Defer non-critical JS** — render skeleton first, hydrate after `screenInteractive`.

## Key Points

- Only cold starts give meaningful TTI; filter prewarmed iOS launches via `ActivePrewarm` env var.
- Set markers across the full pipeline: native init → JS bundle load → content appeared → screen interactive.
- JS bundle parse/eval dominates cold start — bundle size optimizations have the biggest TTI impact.
- `screenInteractive` must mark *interactivity*, not mount — be careful where you call `performance.mark`.
- Target sub-2s TTI; anything over 4s needs work.
- Filter background launches (push notifications) before reporting metrics.

<!-- Source references: https://github.com/callstackincubator/agent-skills/blob/main/skills/react-native-best-practices/references/native-measure-tti.md -->
