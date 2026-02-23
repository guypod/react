# Direct Property Mutation Detection

A utility module that applies read-only tracking to an object and detects when a tracked property is directly mutated. Each mutation triggers a violation log and the write still takes effect so the object continues functioning normally.

## Capabilities

### Detect and log direct property writes on tracked objects

After read-only tracking is applied to an object, any direct write to one of its own writable properties should trigger the violation logger. The logger receives the violation type identifier for a direct mutation, the source string, the property key, and the new value. The write itself succeeds — the property value is updated.

- Writing a new value to a tracked property triggers the logger once with the key and new value [@test](./tests/solution.test.ts)
- After the mutation is logged, reading the property returns the new (mutated) value [@test](./tests/solution.test.ts)
- Writing to the same property twice triggers the logger twice, once per write [@test](./tests/solution.test.ts)
- Writing to an untracked object does not trigger the logger [@test](./tests/solution.test.ts)

## Implementation

[@generates](./src/solution.ts)

## API

```typescript { #api }
/**
 * Marks an object as read-only. Direct writes to its properties will be logged.
 * The writes still succeed — the property values are updated normally.
 * @param obj - The object to track
 * @param source - A source identifier forwarded to the logger on each violation
 * @param logger - Called with (violationType, source, key, newValue) on each property write
 * @returns The same object reference
 */
export function watchMutations<T extends object>(
  obj: T,
  source: string,
  logger: (violation: string, source: string, key: string, value?: unknown) => void,
): T;
```

## Dependencies { .dependencies }

### make-read-only-util 0.0.1 { .dependency }

Provides a factory function returning a tracking function. Once an object is passed to the
tracking function, subsequent writes to its properties are intercepted and reported to the
logger while the write still takes effect in the internal backing store.

[@satisfied-by](make-read-only-util)
