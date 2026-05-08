---
name: react-native-debugging
description: Debugging React Native apps with React Native DevTools, LogBox, Dev Menu, and profiling. Covers RN 0.73+ (Flipper removed, built-in DevTools added).
---

# Debugging

React Native provides built-in debugging tools. Since **RN 0.73**, Flipper is no longer included — the new **React Native DevTools** (built into RN 0.76+) covers all core debugging needs.

---

## Dev Menu

Access the developer overlay for debugging actions.

| Platform | Shortcut |
|----------|---------|
| iOS Simulator | `Ctrl + Cmd + Z` (or Device → Shake) |
| Android Emulator | `Cmd + M` (macOS) / `Ctrl + M` (Win/Linux) |
| Android physical device | `adb shell input keyevent 82` |
| Both (Metro CLI) | `d` in terminal |

**Dev Menu options:**
- **Reload** — reload JS bundle
- **Open DevTools** — opens React Native DevTools
- **Toggle Perf Monitor** — overlay showing FPS and RAM
- **Inspect Element** — tap to inspect component in DevTools
- **Enable Fast Refresh** — auto-reload on file save (on by default)

---

## React Native DevTools (RN 0.76+)

The built-in debugging experience, replacing Flipper. No additional installation needed.

**Open**: Dev Menu → "Open DevTools" (or press `j` in Metro terminal)

**Panels:**

| Panel | Purpose |
|-------|---------|
| **Console** | View logs, run expressions against the live runtime |
| **Components** | Inspect React component tree and props/state |
| **Profiler** | Record renders, find what caused re-renders and how long they took |
| **Sources** | Step-through JS debugger with breakpoints |
| **Network** | Inspect fetch/XHR requests and responses |

### Breakpoints

```ts
// Programmatic breakpoint — pauses in DevTools Sources panel
debugger;

// Or set directly in Sources panel via line number gutter
```

---

## LogBox

In-app error and warning overlay in development builds.

| Type | Appearance |
|------|-----------|
| Fatal error | Full-screen red overlay — app non-functional until fixed |
| Console error | Red badge in corner with count |
| Warning | Yellow banner with prompt to open DevTools |

### Silence Known Warnings

```tsx
import { LogBox } from 'react-native';

// Ignore specific warnings by pattern
LogBox.ignoreLogs([
  'Warning: componentWillReceiveProps',
  'Non-serializable values were found in the navigation state',
]);

// Suppress all warnings (not recommended — masks real issues)
LogBox.ignoreAllLogs();
```

---

## Fast Refresh

Enabled by default — updates only the changed module without losing component state. For changes to root component or native modules, a full reload is required.

Toggle via Dev Menu or:
```bash
r   # in Metro terminal — full reload
d   # open Dev Menu
```

---

## Console Logging

```tsx
console.log('Value:', someValue);      // general output
console.warn('Potential issue');       // yellow in LogBox
console.error('Something failed');     // red in LogBox
console.table(arrayOfObjects);         // tabular format in DevTools
console.group('Section'); /* ... */ console.groupEnd(); // collapsible group
```

Remove logs in production via Babel:
```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

---

## Inspecting Component State

In React Native DevTools → Components panel:
- Click any component in the tree to see its props and state
- Right-click → "Store as global variable" to use in Console
- Search components by name with `Ctrl/Cmd + F`

---

## Network Debugging

React Native DevTools Network panel shows all `fetch` and `XMLHttpRequest` calls. For React Query, the TanStack Query Devtools overlay can also be enabled in development:

```tsx
// Only renders in development
if (__DEV__) {
  const { DevtoolsToggle } = require('@tanstack/react-query-devtools/production');
}
```

---

## Android-Specific: adb

```bash
# Stream device logs filtered by tag
adb logcat *:S ReactNative:V ReactNativeJS:V

# Forward Metro port to device
adb reverse tcp:8081 tcp:8081

# Install and launch APK
adb install app-release.apk
adb shell am start -n com.myapp/.MainActivity
```

---

## iOS-Specific: Console.app

1. Connect device, open **Console.app** on Mac
2. Select device in sidebar
3. Filter by process name or "ReactNative"

Useful for catching native crashes not surfaced in the JS layer.

---

## Common Debugging Patterns

### Track Re-renders

```tsx
import { useRef, useEffect } from 'react';

function useWhyDidYouUpdate(name: string, props: Record<string, unknown>) {
  const prevProps = useRef(props);
  useEffect(() => {
    const changes: Record<string, { from: unknown; to: unknown }> = {};
    Object.keys(props).forEach((key) => {
      if (prevProps.current[key] !== props[key]) {
        changes[key] = { from: prevProps.current[key], to: props[key] };
      }
    });
    if (Object.keys(changes).length) {
      console.log(`[${name}] re-rendered due to:`, changes);
    }
    prevProps.current = props;
  });
}
```

### Measure Render Time

```tsx
console.time('MyComponent render');
// ... component tree
console.timeEnd('MyComponent render');
```

### Identify Slow Operations

```tsx
import { InteractionManager } from 'react-native';

// Log if screen transition is slow
const start = Date.now();
InteractionManager.runAfterInteractions(() => {
  console.log(`Transition took ${Date.now() - start}ms`);
});
```
