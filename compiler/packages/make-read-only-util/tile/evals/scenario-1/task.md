# Primitive Value Pass-Through

A utility module that applies mutation tracking to arbitrary values and demonstrates that primitive values (numbers, booleans, strings, and null) are returned unchanged without triggering any violation logs.

## Capabilities

### Return primitive values unchanged when applying read-only tracking

When the read-only tracking function is applied to a primitive value, it must return that exact value without modification and without calling the violation logger.

- Applying the tracker to a number (e.g., `42`) returns `42` and does not call the logger [@test](./tests/solution.test.ts)
- Applying the tracker to a boolean (e.g., `true`) returns `true` and does not call the logger [@test](./tests/solution.test.ts)
- Applying the tracker to `null` returns `null` and does not call the logger [@test](./tests/solution.test.ts)
- Applying the tracker to a string (e.g., `"hello"`) returns `"hello"` and does not call the logger [@test](./tests/solution.test.ts)

## Implementation

[@generates](./src/solution.ts)

## API

```typescript { #api }
/**
 * Applies read-only tracking to a value.
 * Primitive values are returned unchanged without triggering any violation log.
 * @param value - The value to track
 * @param source - A string identifier for the tracking source
 * @returns The value, unchanged
 */
export function trackValue<T>(value: T, source: string): T;
```

## Dependencies { .dependencies }

### make-read-only-util 0.0.1 { .dependency }

Provides a factory function that returns a tracking function. When called with a primitive value
or null, the tracking function returns the value as-is without any side effects.

[@satisfied-by](make-read-only-util)
