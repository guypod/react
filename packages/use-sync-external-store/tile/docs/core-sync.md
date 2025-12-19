# Core Store Synchronization

The `useSyncExternalStore` hook subscribes to external mutable stores with guaranteed consistency across concurrent renders, preventing tearing where different components see different store values during a single render.

## Capabilities

### useSyncExternalStore Hook

Subscribes to an external data source and returns its current value.

```javascript { .api }
/**
 * Subscribe to an external store with guaranteed consistency.
 *
 * @param subscribe - Function that registers a callback for store changes
 * @param getSnapshot - Function that returns the current store value
 * @param getServerSnapshot - Optional function that returns the server snapshot for SSR
 * @returns Current value from the store
 */
function useSyncExternalStore<T>(
  subscribe: (callback: () => void) => () => void,
  getSnapshot: () => T,
  getServerSnapshot?: () => T
): T;
```

**Parameters**:

- `subscribe` - Function that registers a callback to be called whenever the store changes. Must return an unsubscribe function.
  - The callback should be invoked whenever the store value changes
  - The returned function will be called to unsubscribe when the component unmounts or the subscription changes
  - Type: `(callback: () => void) => () => void`

- `getSnapshot` - Function that returns the current value from the store. Must return a stable/cached reference when the value hasn't changed.
  - Called on every render and during subscription changes
  - Should return the same reference (by `Object.is`) if the value hasn't changed
  - **Important**: The result must be cached/memoized to avoid infinite loops
  - Type: `() => T`

- `getServerSnapshot` (optional) - Function that returns the initial snapshot for server-side rendering
  - Used during SSR to provide the server-rendered value
  - Ensures hydration consistency between server and client
  - **Note**: In the shim for React <18, this parameter is accepted but not used because those React versions don't expose hydration state
  - Type: `() => T`

**Returns**: The current snapshot value from the store (type `T`)

## Usage Examples

### Basic Store Subscription

```javascript
import { useSyncExternalStore } from 'use-sync-external-store/shim';

// Simple counter store
let count = 0;
const listeners = new Set();

const counterStore = {
  subscribe(callback) {
    listeners.add(callback);
    return () => listeners.delete(callback);
  },
  getSnapshot() {
    return count;
  },
  increment() {
    count++;
    listeners.forEach(listener => listener());
  }
};

function Counter() {
  const value = useSyncExternalStore(
    counterStore.subscribe,
    counterStore.getSnapshot
  );

  return (
    <div>
      <p>Count: {value}</p>
      <button onClick={counterStore.increment}>Increment</button>
    </div>
  );
}
```

### Object Store with Memoization

```javascript
import { useSyncExternalStore } from 'use-sync-external-store/shim';

class Store {
  constructor(initialState) {
    this.state = initialState;
    this.listeners = new Set();
    this.cachedSnapshot = initialState;
  }

  subscribe = (callback) => {
    this.listeners.add(callback);
    return () => this.listeners.delete(callback);
  };

  // Cached getSnapshot to avoid infinite loops
  getSnapshot = () => {
    return this.cachedSnapshot;
  };

  setState(newState) {
    this.state = newState;
    this.cachedSnapshot = newState; // Update cache
    this.listeners.forEach(listener => listener());
  }
}

const userStore = new Store({ name: 'Alice', age: 25 });

function UserProfile() {
  const user = useSyncExternalStore(
    userStore.subscribe,
    userStore.getSnapshot
  );

  return (
    <div>
      <p>Name: {user.name}</p>
      <p>Age: {user.age}</p>
    </div>
  );
}
```

### Browser API Subscription (Online Status)

```javascript
import { useSyncExternalStore } from 'use-sync-external-store/shim';

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function getSnapshot() {
  return navigator.onLine;
}

function getServerSnapshot() {
  // Assume online during SSR
  return true;
}

function OnlineStatus() {
  const isOnline = useSyncExternalStore(
    subscribe,
    getSnapshot,
    getServerSnapshot
  );

  return <div>Status: {isOnline ? 'Online' : 'Offline'}</div>;
}
```

### Server-Side Rendering Support

```javascript
import { useSyncExternalStore } from 'use-sync-external-store/shim';

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
    getServerSnapshot() {
      // Return initial/default state during SSR
      return initialState;
    },
    setState(newState) {
      state = newState;
      listeners.forEach(listener => listener());
    }
  };
}

const store = createStore({ items: [] });

function ItemList() {
  const state = useSyncExternalStore(
    store.subscribe,
    store.getSnapshot,
    store.getServerSnapshot  // Ensures consistent SSR hydration
  );

  return (
    <ul>
      {state.items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}
```

### Redux Integration Example

```javascript
import { useSyncExternalStore } from 'use-sync-external-store/shim';
import { createStore } from 'redux';

const reduxStore = createStore(reducer);

function useReduxStore() {
  return useSyncExternalStore(
    reduxStore.subscribe,
    reduxStore.getState,
    reduxStore.getState
  );
}

function App() {
  const state = useReduxStore();
  return <div>{/* Use Redux state */}</div>;
}
```

## Important Considerations

### getSnapshot Must Be Cached

The `getSnapshot` function must return a cached/stable reference when the value hasn't changed. Returning a new reference on every call causes infinite loops:

```javascript
// ❌ BAD - Creates new object every call
function getSnapshot() {
  return { value: count };  // New object reference each time
}

// ✅ GOOD - Returns cached reference
let cachedSnapshot = { value: 0 };
function getSnapshot() {
  return cachedSnapshot;
}
function updateCount(newValue) {
  cachedSnapshot = { value: newValue };  // Update cache
  notifyListeners();
}
```

### Synchronous Updates Required

The store must notify subscribers synchronously. Asynchronous notifications can cause inconsistencies:

```javascript
// ❌ BAD - Async notification
function setState(newState) {
  state = newState;
  setTimeout(() => {
    listeners.forEach(listener => listener());
  }, 0);
}

// ✅ GOOD - Sync notification
function setState(newState) {
  state = newState;
  listeners.forEach(listener => listener());
}
```

### React Native Considerations

React Native apps should use the shim entry point, which automatically resolves to the React Native-specific implementation:

```javascript
import { useSyncExternalStore } from 'use-sync-external-store/shim';
// Automatically uses shim/index.native.js on React Native
```

### SSR and Hydration (React <18)

When using the shim with React versions below 18, the `getServerSnapshot` parameter is not used because these versions don't expose hydration state. Instead, `getSnapshot` must return the appropriate value based on the environment:

```javascript
function getSnapshot() {
  // Manual SSR handling for React <18
  if (typeof window === 'undefined') {
    return defaultServerValue;
  }
  return currentClientValue;
}
```

For React 18+, use the `getServerSnapshot` parameter for proper SSR support.
