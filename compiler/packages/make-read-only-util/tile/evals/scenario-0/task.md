# Read-Only Tracker Factory Setup

A utility module that creates a read-only mutation tracker using the `make-read-only-util` package. The module initializes the tracker with a logger callback and a list of class names to skip, then exports the resulting tracking function.

## Capabilities

### Initialize a read-only tracker with a logger and skip list

Create and export a function that tracks mutations on read-only objects. The tracker is produced by calling the package's factory function with a violation logger and an array of class names whose instances should be excluded from tracking.

- Given a logger function and an empty skip list, the factory returns a tracking function that can be applied to any object [@test](./tests/solution.test.ts)
- Given a logger function and a non-empty skip list (e.g., `["Map", "Set"]`), the factory returns a working tracking function [@test](./tests/solution.test.ts)
- The returned tracking function accepts an object and a source string, and returns the same object [@test](./tests/solution.test.ts)

## Implementation

[@generates](./src/solution.ts)

## API

```typescript { #api }
/**
 * Creates a mutation tracker.
 * @param logger - Called whenever a violation is detected
 * @param skippedClasses - Constructor names whose instances are excluded from tracking
 * @returns A function that marks objects as read-only and tracks mutations
 */
export function createTracker(
  logger: (violation: string, source: string, key: string, value?: unknown) => void,
  skippedClasses: string[],
): <T>(value: T, source: string) => T;
```

## Dependencies { .dependencies }

### make-read-only-util 0.0.1 { .dependency }

Provides a factory function that accepts a violation logger and a list of class names to skip,
and returns a function for marking objects as read-only and tracking mutations.

[@satisfied-by](make-read-only-util)
