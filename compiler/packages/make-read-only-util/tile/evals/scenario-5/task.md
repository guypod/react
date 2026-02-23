# Transitive Nested Mutation Tracking

A utility module that applies read-only tracking to an object with nested object-valued properties. When a property on a nested object is mutated, the violation logger is triggered — tracking propagates through the object graph transitively.

## Capabilities

### Detect mutations on deeply nested properties of a tracked object

When a top-level object is marked as read-only, mutations to properties on any of its nested object values are also tracked. The nested object does not need to be explicitly registered; tracking is applied lazily as nested objects are accessed through the tracked parent.

- Mutating a property on a nested object (e.g., `parent.child.x = 1`) triggers the logger [@test](./tests/solution.test.ts)
- The logger is called with the key of the mutated nested property and the new value [@test](./tests/solution.test.ts)
- The top-level source string is included in the logger call for nested mutations [@test](./tests/solution.test.ts)

## Implementation

[@generates](./src/solution.ts)

## API

```typescript { #api }
/**
 * Marks an object and its reachable nested objects as read-only.
 * Mutations anywhere in the object graph trigger the violation logger.
 * @param obj - The root object to track
 * @param source - A source identifier forwarded to the logger on each violation
 * @param logger - Called with (violationType, source, key, newValue) on any mutation
 * @returns The same root object reference
 */
export function watchDeep<T extends object>(
  obj: T,
  source: string,
  logger: (violation: string, source: string, key: string, value?: unknown) => void,
): T;
```

## Dependencies { .dependencies }

### make-read-only-util 0.0.1 { .dependency }

Provides a factory function returning a tracking function. When an object is tracked, its
nested object-valued properties are tracked lazily on access, so that mutations at any depth
in the object graph trigger the logger with the originally provided source string.

[@satisfied-by](make-read-only-util)
