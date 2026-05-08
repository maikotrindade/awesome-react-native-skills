---
name: react-native-timers
description: JavaScript timers and scheduling APIs. Use when implementing delays, intervals, or scheduling work after interactions.
---

# Timers

React Native implements standard browser timers with platform-specific optimizations.

## Standard Timers

### setTimeout / clearTimeout

Execute code after a delay.

```tsx
// Set timeout
const timeoutId = setTimeout(() => {
  console.log('Executed after 1 second');
}, 1000);

// Clear timeout
clearTimeout(timeoutId);
```

### setInterval / clearInterval

Execute code repeatedly at intervals.

```tsx
// Set interval
const intervalId = setInterval(() => {
  console.log('Executed every second');
}, 1000);

// Clear interval
clearInterval(intervalId);
```

### React Hook Pattern

```tsx
import { useEffect, useRef } from 'react';

const App = () => {
  const timeoutRef = useRef<NodeJS.Timeout>();

  useEffect(() => {
    timeoutRef.current = setTimeout(() => {
      console.log('Delayed execution');
    }, 1000);

    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  return null;
};
```

## Animation Timers

### requestAnimationFrame / cancelAnimationFrame

Execute code before next repaint (for animations).

```tsx
const animationFrameId = requestAnimationFrame(() => {
  // Update animation
  updateAnimation();
  
  // Schedule next frame
  requestAnimationFrame(updateAnimation);
});

// Cancel
cancelAnimationFrame(animationFrameId);
```

**Note**: `requestAnimationFrame` fires after all frames flush, not immediately like `setTimeout(fn, 0)`.

### Animation Loop Pattern

```tsx
const useAnimationFrame = (callback: () => void) => {
  const frameRef = useRef<number>();

  useEffect(() => {
    const animate = () => {
      callback();
      frameRef.current = requestAnimationFrame(animate);
    };

    frameRef.current = requestAnimationFrame(animate);

    return () => {
      if (frameRef.current) {
        cancelAnimationFrame(frameRef.current);
      }
    };
  }, [callback]);
};
```

## Immediate Execution

### setImmediate / clearImmediate

Execute code at end of current execution block.

```tsx
setImmediate(() => {
  console.log('Executed after current block');
});
```

**Use cases:**
- Defer work until after current render
- Batch operations
- Avoid blocking current execution

**Note**: `InteractionManager.runAfterInteractions()` is deprecated in favor of `setImmediate`.

## InteractionManager (Deprecated)

Schedule work after interactions/animations complete.

```tsx
import { InteractionManager } from 'react-native';

// Deprecated - use setImmediate instead
InteractionManager.runAfterInteractions(() => {
  // Heavy work after interactions
  loadData();
});
```

**Replacement**: Use `setImmediate` or `requestIdleCallback`.

## Timer Comparison

| Timer | Use Case | Timing |
|-------|----------|--------|
| `setTimeout` | Delayed execution | After specified delay |
| `setInterval` | Repeated execution | At specified intervals |
| `requestAnimationFrame` | Animation updates | Before next repaint (~60fps) |
| `setImmediate` | Defer to next tick | End of current block |

## Common Patterns

### Debounce

```tsx
const useDebounce = (value: string, delay: number) => {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
};
```

### Throttle

```tsx
const useThrottle = (callback: () => void, delay: number) => {
  const lastRun = useRef(Date.now());

  return useCallback(() => {
    if (Date.now() - lastRun.current >= delay) {
      callback();
      lastRun.current = Date.now();
    }
  }, [callback, delay]);
};
```

### Polling

```tsx
const usePolling = (callback: () => void, interval: number) => {
  useEffect(() => {
    const intervalId = setInterval(callback, interval);

    return () => {
      clearInterval(intervalId);
    };
  }, [callback, interval]);
};
```

### Delayed State Update

```tsx
const [visible, setVisible] = useState(false);

useEffect(() => {
  const timeout = setTimeout(() => {
    setVisible(true);
  }, 1000);

  return () => clearTimeout(timeout);
}, []);
```

## Best Practices

1. **Clean up timers** - Always clear in useEffect cleanup
2. **Use refs for IDs** - Store timer IDs in refs, not state
3. **Prefer requestAnimationFrame** - For animation updates
4. **Use setImmediate** - Instead of deprecated InteractionManager
5. **Debounce/throttle** - For frequent events (scroll, input)
6. **Test cleanup** - Ensure timers are cleared on unmount
7. **Avoid memory leaks** - Clear all timers in cleanup

## Common Patterns

### Auto-hide Message

```tsx
const [message, setMessage] = useState('');

const showMessage = (text: string) => {
  setMessage(text);
  setTimeout(() => {
    setMessage('');
  }, 3000);
};
```

### Retry with Backoff

```tsx
const retryWithBackoff = async (
  fn: () => Promise<void>,
  maxRetries = 3
) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await fn();
      return;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, 1000 * Math.pow(2, i))
      );
    }
  }
};
```

<!--
Source references:
- https://reactnative.dev/docs/timers
- https://reactnative.dev/docs/interactionmanager
-->
