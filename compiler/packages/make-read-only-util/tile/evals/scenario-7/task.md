# Cyclic Reference Handling

A utility module that applies read-only tracking to objects containing circular references. The tracking function must handle cyclic object graphs without entering infinite recursion or throwing an error.

## Capabilities

### Handle objects with circular references without infinite recursion

When read-only tracking is applied to an object that contains a direct or indirect reference back to itself, the function returns normally. The same object can be passed to the tracking function multiple times without causing a stack overflow or error.

- Applying tracking to an object with a self-referential property (e.g., `obj.self = obj`) returns normally [@test](./tests/solution.test.ts)
- The returned value from tracking a self-referential object is the same object reference (`===`) [@test](./tests/solution.test.ts)
- Accessing the self-referential property after tracking still returns the same object [@test](./tests/solution.test.ts)

## Implementation

[@generates](./src/solution.ts)

## API

```typescript { #api }
/**
 * Applies read-only tracking to an object, safely handling circular references.
 * @param obj - The object to track (may contain circular references)
 * @param source - A source identifier for the tracking registration
 * @param logger - Called with (violationType, source, key, value?) on each violation
 * @returns The same object reference
 */
export function trackCyclic<T extends object>(
  obj: T,
  source: string,
  logger: (violation: string, source: string, key: string, value?: unknown) => void,
): T;
```

## Dependencies { .dependencies }

### make-read-only-util 0.0.1 { .dependency }

Provides a factory function returning a tracking function that uses a WeakMap-based cache
to record already-visited objects. This cache prevents infinite recursion when the package
encounters circular references during tracking.

[@satisfied-by](make-read-only-util)
