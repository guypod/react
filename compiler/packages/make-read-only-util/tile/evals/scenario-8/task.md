# Property Addition Detection Between Calls

A utility module that uses read-only tracking to detect when new properties are added to a previously tracked object. The detection requires calling the tracking function on the object a second time after the addition.

## Capabilities

### Detect new properties added to a tracked object on re-evaluation

When the tracking function is called on an object for the first time, the object's current properties are registered in the cache. If a new property is added to the object and the tracking function is called again on the same object, the new property is detected and the logger is called to report the addition.

- Adding a property to a tracked object and re-applying tracking causes the logger to be called for the added property [@test](./tests/solution.test.ts)
- The logger is called with the key of the newly added property [@test](./tests/solution.test.ts)
- Properties that existed when tracking was first applied do not trigger an addition log on re-evaluation [@test](./tests/solution.test.ts)

## Implementation

[@generates](./src/solution.ts)

## API

```typescript { #api }
/**
 * Applies read-only tracking to an object. On the second and subsequent calls
 * with the same object, any properties added since the last call are logged as additions.
 * @param obj - The object to track
 * @param source - A source identifier for the tracking registration
 * @param logger - Called with (violationType, source, key) when an added property is detected
 * @returns The same object reference
 */
export function trackWithAdditionDetection<T extends object>(
  obj: T,
  source: string,
  logger: (violation: string, source: string, key: string, value?: unknown) => void,
): T;
```

## Dependencies { .dependencies }

### make-read-only-util 0.0.1 { .dependency }

Provides a factory function returning a tracking function. On the second call to makeReadOnly
with the same object, any properties that were added between the two calls are detected and
reported to the logger with the addition violation type.

[@satisfied-by](make-read-only-util)
