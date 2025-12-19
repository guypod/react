# use-sync-external-store

A backwards-compatible shim for React's `useSyncExternalStore` hook that enables safe subscription to external mutable stores across React 16.8+, 17.x, and 18+. This package provides both the core synchronization primitive and an optimized selector API for partial store subscriptions, with automatic fallback to polyfill implementations when the native React 18+ API is unavailable.

## Package Information

- **Package Name**: use-sync-external-store
- **Package Type**: npm
- **Language**: JavaScript
- **Installation**: `npm install use-sync-external-store`

## Core Imports

### For React 18+ Projects (Native API)

```javascript
import { useSyncExternalStore } from 'use-sync-external-store';
```

**Note**: This entry point only works with React 18+ as it directly re-exports from React. For backwards compatibility with React 16.8+ and 17.x, use the shim entry point below.

### For React 16.8+ / 17.x / 18+ Projects (Backwards Compatible Shim)

```javascript
import { useSyncExternalStore } from 'use-sync-external-store/shim';
```

The shim automatically detects the React version and uses the native implementation when available (React 18+) or falls back to a polyfill for older versions.

### With Selector API

```javascript
import { useSyncExternalStoreWithSelector } from 'use-sync-external-store/with-selector';
// Or for backwards compatibility:
import { useSyncExternalStoreWithSelector } from 'use-sync-external-store/shim/with-selector';
```

CommonJS:

```javascript
const { useSyncExternalStore } = require('use-sync-external-store/shim');
const { useSyncExternalStoreWithSelector } = require('use-sync-external-store/shim/with-selector');
```

## Basic Usage

```javascript
import { useSyncExternalStore } from 'use-sync-external-store/shim';
import { createStore } from './my-store';

// Create an external store
const store = createStore({ count: 0 });

function Counter() {
  // Subscribe to the store
  const state = useSyncExternalStore(
    store.subscribe,    // Subscribe function
    store.getSnapshot,  // Get current value
    store.getSnapshot   // Get server snapshot (for SSR)
  );

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => store.setState({ count: state.count + 1 })}>
        Increment
      </button>
    </div>
  );
}
```

Example store implementation:

```javascript
function createStore(initialState) {
  let state = initialState;
  let listeners = new Set();

  return {
    subscribe(callback) {
      listeners.add(callback);
      return () => listeners.delete(callback);
    },
    getSnapshot() {
      return state;
    },
    setState(newState) {
      state = newState;
      listeners.forEach(listener => listener());
    }
  };
}
```

## Architecture

The package provides multiple entry points and implementations:

- **Main Entry** (`use-sync-external-store`): Direct re-export from React 18+, requires React 18 or higher
- **Shim Entry** (`use-sync-external-store/shim`): Backwards-compatible implementation that auto-detects React version
- **Selector Variants** (`with-selector` and `shim/with-selector`): Enhanced API with selector and custom equality support
- **Platform Variants**: Separate implementations for web, React Native, and server-side rendering

The shim implementation uses React's built-in API when available (React 18+) or provides a polyfill that leverages `useState`, `useEffect`, and `useLayoutEffect` to maintain synchronous behavior in older React versions.

## Capabilities

### Core Store Synchronization

Subscribe to external mutable stores with guaranteed consistency across concurrent renders, preventing "tearing" where different components see different values from the same store during a single render.

```javascript { .api }
function useSyncExternalStore<T>(
  subscribe: (callback: () => void) => () => void,
  getSnapshot: () => T,
  getServerSnapshot?: () => T
): T;
```

[Core Store Synchronization](./core-sync.md)

### Optimized Partial Store Subscriptions

Subscribe to derived values from stores with custom selector and equality functions, enabling fine-grained subscriptions that only trigger re-renders when selected data actually changes.

```javascript { .api }
function useSyncExternalStoreWithSelector<Snapshot, Selection>(
  subscribe: (callback: () => void) => () => void,
  getSnapshot: () => Snapshot,
  getServerSnapshot: void | null | (() => Snapshot),
  selector: (snapshot: Snapshot) => Selection,
  isEqual?: (a: Selection, b: Selection) => boolean
): Selection;
```

[Optimized Partial Store Subscriptions](./with-selector.md)

## Entry Points Reference

| Entry Point | Description | React Version |
|-------------|-------------|---------------|
| `use-sync-external-store` | Native API re-export | React 18+ only |
| `use-sync-external-store/shim` | Backwards-compatible shim | React 16.8+, 17.x, 18+ |
| `use-sync-external-store/with-selector` | Selector API | React 18+ only |
| `use-sync-external-store/shim/with-selector` | Selector shim | React 16.8+, 17.x, 18+ |
| `use-sync-external-store/shim/index.native.js` | React Native shim | React Native 16.8+ |

## Platform Support

- **Web**: Fully supported via main and shim entry points
- **React Native**: Use `use-sync-external-store/shim` which automatically resolves to `shim/index.native.js`
- **Server-Side Rendering**: Supported via `getServerSnapshot` parameter (React 18+) or manual hydration state management (React <18)
