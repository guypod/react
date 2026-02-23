# Property Deletion Detection Between Calls

A utility module that uses read-only tracking to detect when properties are deleted from a previously tracked object. The detection requires calling the tracking function on the object a second time after the deletion.

## Capabilities

### Detect properties deleted from a tracked object on re-evaluation

When the tracking function is called on an object for the first time, its properties are registered. If a property is subsequently deleted and the tracking function is called again on the same object, the deletion is detected and the logger is called.

- Deleting a property from a tracked object and re-applying tracking causes the logger to be called for the deleted property [@test](./tests/solution.test.ts)
- The logger is called with the key of the deleted property [@test](./tests/solution.test.ts)
- Properties that still exist when re-tracking is applied do not trigger a deletion log [@test](./tests/solution.test.ts)

## Implementation

[@generates](./src/solution.ts)

## API

```typescript { #api }
/**
 * Applies read-only tracking to an object. On the second and subsequent calls
 * with the same object, any properties deleted since the last call are logged as deletions.
 * @param obj - The object to track
 * @param source - A source identifier for the tracking registration
 * @param logger - Called with (violationType, source, key) when a deleted property is detected
 * @returns The same object reference
 */
export function trackWithDeletionDetection<T extends object>(
  obj: T,
  source: string,
  logger: (violation: string, source: string, key: string, value?: unknown) => void,
): T;
```

## Dependencies { .dependencies }

### make-read-only-util 0.0.1 { .dependency }

Provides a factory function returning a tracking function. On the second call to makeReadOnly
with the same object, any properties that were deleted between the two calls are detected and
reported to the logger with the deletion violation type.

[@satisfied-by](make-read-only-util)
