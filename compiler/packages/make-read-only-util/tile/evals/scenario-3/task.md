# Source Identifier in Violation Logs

A utility module that applies read-only tracking to objects with a caller-supplied source identifier string. When a mutation violation is detected, the logger is called with the source string so that the violation can be traced back to the code site that registered the constraint.

## Capabilities

### Include the source identifier in every violation log call

The read-only tracking function accepts a source string that must be forwarded to the violation logger on every violation event. The source enables callers to identify which registration site a violation originates from.

- When a property is mutated on a tracked object, the logger is called with the exact source string that was passed to the tracking function [@test](./tests/solution.test.ts)
- When a different source string is used for a different object, violations from that object are logged with the corresponding source string [@test](./tests/solution.test.ts)
- The source string appears as the second argument to the logger callback (after the violation type) [@test](./tests/solution.test.ts)

## Implementation

[@generates](./src/solution.ts)

## API

```typescript { #api }
/**
 * Marks an object as read-only with a source identifier.
 * Any detected mutation on the object will be reported to the logger
 * along with the provided source string.
 * @param obj - The object to track
 * @param source - An identifier string included in every violation log call
 * @param logger - Called with (violationType, source, key, value?) on each violation
 * @returns The same object reference
 */
export function trackWithSource<T extends object>(
  obj: T,
  source: string,
  logger: (violation: string, source: string, key: string, value?: unknown) => void,
): T;
```

## Dependencies { .dependencies }

### make-read-only-util 0.0.1 { .dependency }

Provides a factory function that binds a logger and produces a tracking function. The tracking
function accepts both the object to track and a source string; the source string is forwarded
to the logger on every violation notification.

[@satisfied-by](make-read-only-util)
