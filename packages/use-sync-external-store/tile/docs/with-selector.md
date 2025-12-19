# Optimized Partial Store Subscriptions

The `useSyncExternalStoreWithSelector` hook extends `useSyncExternalStore` with selector and custom equality support, enabling fine-grained subscriptions that only trigger re-renders when selected data changes. This is ideal for subscribing to partial state from large stores.

## Capabilities

### useSyncExternalStoreWithSelector Hook

Subscribes to a derived value from an external store with custom selector and equality functions.

```javascript { .api }
/**
 * Subscribe to a derived value from an external store with custom selector and equality.
 *
 * @param subscribe - Function that registers a callback for store changes
 * @param getSnapshot - Function that returns the current store value
 * @param getServerSnapshot - Optional function that returns the server snapshot for SSR
 * @param selector - Function that derives a value from the snapshot
 * @param isEqual - Optional custom equality function for comparing selections
 * @returns Selected value from the store
 */
function useSyncExternalStoreWithSelector<Snapshot, Selection>(
  subscribe: (callback: () => void) => () => void,
  getSnapshot: () => Snapshot,
  getServerSnapshot: void | null | (() => Snapshot),
  selector: (snapshot: Snapshot) => Selection,
  isEqual?: (a: Selection, b: Selection) => boolean
): Selection;
```

**Parameters**:

- `subscribe` - Function that registers a callback to be called whenever the store changes. Must return an unsubscribe function.
  - Same as `useSyncExternalStore`
  - Type: `(callback: () => void) => () => void`

- `getSnapshot` - Function that returns the current value from the store.
  - Same as `useSyncExternalStore`
  - Returns the full store snapshot
  - Type: `() => Snapshot`

- `getServerSnapshot` - Function that returns the initial snapshot for server-side rendering
  - Can be `null`, `undefined`, or a function
  - Used during SSR (React 18+)
  - Type: `void | null | (() => Snapshot)`

- `selector` - Function that extracts a derived value from the snapshot
  - Called with the current snapshot to compute the selected value
  - Enables subscribing to only part of the store
  - Should be a pure function
  - Type: `(snapshot: Snapshot) => Selection`

- `isEqual` (optional) - Custom equality function for comparing selected values
  - If not provided, uses `Object.is` comparison
  - Return `true` if values are equal, `false` otherwise
  - Prevents re-renders when selected values are conceptually equal even if references differ
  - Useful for deep equality checks or custom comparison logic
  - Type: `(a: Selection, b: Selection) => boolean`

**Returns**: The selected value from the store (type `Selection`)

## Usage Examples

### Basic Selector Usage

```javascript
import { useSyncExternalStoreWithSelector } from 'use-sync-external-store/shim/with-selector';

// Store with multiple properties
const store = {
  state: { user: { name: 'Alice', age: 25 }, theme: 'dark', count: 0 },
  listeners: new Set(),

  subscribe(callback) {
    this.listeners.add(callback);
    return () => this.listeners.delete(callback);
  },

  getSnapshot() {
    return this.state;
  },

  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.listeners.forEach(listener => listener());
  }
};

// Component only subscribes to user.name
function UserName() {
  const name = useSyncExternalStoreWithSelector(
    store.subscribe,
    store.getSnapshot,
    store.getSnapshot,
    (state) => state.user.name  // Selector
  );

  // Only re-renders when user.name changes, not when theme or count change
  return <div>Name: {name}</div>;
}
```

### Selector with Custom Equality

```javascript
import { useSyncExternalStoreWithSelector } from 'use-sync-external-store/shim/with-selector';

// Shallow equality comparison for objects
function shallowEqual(a, b) {
  if (Object.is(a, b)) return true;
  if (typeof a !== 'object' || a === null || typeof b !== 'object' || b === null) {
    return false;
  }
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  if (keysA.length !== keysB.length) return false;
  for (let key of keysA) {
    if (!Object.is(a[key], b[key])) return false;
  }
  return true;
}

function UserProfile() {
  const user = useSyncExternalStoreWithSelector(
    store.subscribe,
    store.getSnapshot,
    store.getSnapshot,
    (state) => state.user,  // Returns a new object reference
    shallowEqual  // Custom equality prevents unnecessary re-renders
  );

  return (
    <div>
      <p>Name: {user.name}</p>
      <p>Age: {user.age}</p>
    </div>
  );
}
```

### Array Filtering with Selector

```javascript
import { useSyncExternalStoreWithSelector } from 'use-sync-external-store/shim/with-selector';

const todoStore = {
  state: {
    todos: [
      { id: 1, text: 'Learn React', completed: true },
      { id: 2, text: 'Learn Redux', completed: false },
      { id: 3, text: 'Build app', completed: false }
    ]
  },
  listeners: new Set(),

  subscribe(callback) {
    this.listeners.add(callback);
    return () => this.listeners.delete(callback);
  },

  getSnapshot() {
    return this.state;
  }
};

// Deep equality for arrays
function deepEqual(a, b) {
  return JSON.stringify(a) === JSON.stringify(b);
}

function ActiveTodos() {
  const activeTodos = useSyncExternalStoreWithSelector(
    todoStore.subscribe,
    todoStore.getSnapshot,
    todoStore.getSnapshot,
    (state) => state.todos.filter(todo => !todo.completed),
    deepEqual  // Prevents re-render if filtered array content is the same
  );

  return (
    <ul>
      {activeTodos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

### Computed Values from Store

```javascript
import { useSyncExternalStoreWithSelector } from 'use-sync-external-store/shim/with-selector';

const cartStore = {
  state: {
    items: [
      { id: 1, name: 'Widget', price: 10, quantity: 2 },
      { id: 2, name: 'Gadget', price: 20, quantity: 1 }
    ]
  },
  listeners: new Set(),

  subscribe(callback) {
    this.listeners.add(callback);
    return () => this.listeners.delete(callback);
  },

  getSnapshot() {
    return this.state;
  }
};

function CartTotal() {
  const total = useSyncExternalStoreWithSelector(
    cartStore.subscribe,
    cartStore.getSnapshot,
    cartStore.getSnapshot,
    (state) => state.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    )
    // No custom isEqual needed for primitive numbers
  );

  return <div>Total: ${total}</div>;
}
```

### Redux Integration with Selector

```javascript
import { useSyncExternalStoreWithSelector } from 'use-sync-external-store/shim/with-selector';
import { createStore } from 'redux';

const reduxStore = createStore(reducer);

function useReduxSelector(selector, equalityFn) {
  return useSyncExternalStoreWithSelector(
    reduxStore.subscribe,
    reduxStore.getState,
    reduxStore.getState,
    selector,
    equalityFn
  );
}

function TodoList() {
  // Only re-renders when todos array changes
  const todos = useReduxSelector(
    state => state.todos,
    (a, b) => a.length === b.length && a.every((item, i) => item === b[i])
  );

  return (
    <ul>
      {todos.map(todo => <li key={todo.id}>{todo.text}</li>)}
    </ul>
  );
}
```

### Zustand-style Store with Selector

```javascript
import { useSyncExternalStoreWithSelector } from 'use-sync-external-store/shim/with-selector';

function createStore(initialState) {
  let state = initialState;
  const listeners = new Set();

  return {
    subscribe(callback) {
      listeners.add(callback);
      return () => listeners.delete(callback);
    },
    getState() {
      return state;
    },
    setState(partial) {
      state = typeof partial === 'function' ? partial(state) : { ...state, ...partial };
      listeners.forEach(listener => listener());
    },
    // Custom hook with selector support
    useStore(selector, equalityFn) {
      return useSyncExternalStoreWithSelector(
        this.subscribe,
        this.getState,
        this.getState,
        selector || (state => state),
        equalityFn
      );
    }
  };
}

const useStore = createStore({
  count: 0,
  user: { name: 'Alice' },
  increment: () => useStore.setState(state => ({ count: state.count + 1 }))
});

function Counter() {
  // Only subscribes to count
  const count = useStore.useStore(state => state.count);
  const increment = useStore.useStore(state => state.increment);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

### Multiple Selectors in One Component

```javascript
import { useSyncExternalStoreWithSelector } from 'use-sync-external-store/shim/with-selector';

function Dashboard() {
  // Each selector independently optimizes its re-renders
  const userName = useSyncExternalStoreWithSelector(
    store.subscribe,
    store.getSnapshot,
    store.getSnapshot,
    state => state.user.name
  );

  const itemCount = useSyncExternalStoreWithSelector(
    store.subscribe,
    store.getSnapshot,
    store.getSnapshot,
    state => state.items.length
  );

  const theme = useSyncExternalStoreWithSelector(
    store.subscribe,
    store.getSnapshot,
    store.getSnapshot,
    state => state.theme
  );

  // Component only re-renders when userName, itemCount, or theme change
  return (
    <div className={theme}>
      <h1>Welcome {userName}</h1>
      <p>You have {itemCount} items</p>
    </div>
  );
}
```

## Performance Benefits

### Without Selector (Always Re-renders)

```javascript
// Re-renders whenever ANY part of the store changes
function Component() {
  const state = useSyncExternalStore(
    store.subscribe,
    store.getSnapshot
  );

  return <div>{state.user.name}</div>;
  // Re-renders even when unrelated state.theme or state.count change
}
```

### With Selector (Optimized Re-renders)

```javascript
// Only re-renders when state.user.name actually changes
function Component() {
  const name = useSyncExternalStoreWithSelector(
    store.subscribe,
    store.getSnapshot,
    store.getSnapshot,
    state => state.user.name
  );

  return <div>{name}</div>;
  // Doesn't re-render when state.theme or state.count change
}
```

## Important Considerations

### Selector Function Stability

The selector function doesn't need to be memoized with `useCallback`. The hook handles selector changes internally:

```javascript
// ✅ GOOD - Inline selector is fine
const name = useSyncExternalStoreWithSelector(
  store.subscribe,
  store.getSnapshot,
  store.getSnapshot,
  state => state.user.name  // Inline function
);

// ⚠️ UNNECESSARY - No need for useCallback
const selector = useCallback(state => state.user.name, []);
const name = useSyncExternalStoreWithSelector(
  store.subscribe,
  store.getSnapshot,
  store.getSnapshot,
  selector
);
```

### Custom Equality for Reference Types

When selector returns objects or arrays, use custom equality to avoid unnecessary re-renders:

```javascript
// ❌ Without custom equality - re-renders on every store change
const user = useSyncExternalStoreWithSelector(
  store.subscribe,
  store.getSnapshot,
  store.getSnapshot,
  state => ({ name: state.user.name, age: state.user.age })
  // No isEqual - uses Object.is, always sees new object reference
);

// ✅ With custom equality - only re-renders when values change
const user = useSyncExternalStoreWithSelector(
  store.subscribe,
  store.getSnapshot,
  store.getSnapshot,
  state => ({ name: state.user.name, age: state.user.age }),
  (a, b) => a.name === b.name && a.age === b.age
);
```

### Server-Side Rendering

The `getServerSnapshot` parameter works the same as in `useSyncExternalStore`:

```javascript
function Component() {
  const value = useSyncExternalStoreWithSelector(
    store.subscribe,
    store.getSnapshot,
    () => store.getInitialState(),  // Server snapshot
    state => state.someValue
  );

  return <div>{value}</div>;
}
```

### Performance vs. useSyncExternalStore

Use `useSyncExternalStoreWithSelector` when:
- Subscribing to large stores where most changes don't affect the component
- Deriving computed values from store state
- Filtering or mapping store data

Use plain `useSyncExternalStore` when:
- Subscribing to the entire store (no partial selection needed)
- Store values are already primitives
- No need for custom equality logic
