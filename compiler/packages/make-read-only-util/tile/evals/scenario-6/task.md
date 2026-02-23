# Class Instance Skipping

A utility module that configures read-only tracking to skip instances of specified classes. Objects whose constructor name appears in the exclusion list are returned from the tracking function unchanged without any property proxies being installed.

## Capabilities

### Exclude instances of named classes from read-only tracking

When the factory function is configured with a list of class names to skip, objects whose `constructor.name` matches any entry in that list are not tracked. Mutations to properties of skipped objects do not trigger the logger, even if tracking is attempted on those objects.

- An instance of a class whose name is in the skip list is returned unchanged by the tracking function [@test](./tests/solution.test.ts)
- Mutating a property on a skipped class instance does not call the logger [@test](./tests/solution.test.ts)
- An instance of a class whose name is NOT in the skip list is still tracked normally [@test](./tests/solution.test.ts)

## Implementation

[@generates](./src/solution.ts)

## API

```typescript { #api }
/**
 * Creates a read-only tracker that excludes instances of specified classes.
 * @param skippedClassNames - Constructor names of classes to exclude from tracking
 * @param logger - Called with (violationType, source, key, value?) on each violation
 * @returns A tracking function that skips instances of the listed classes
 */
export function createSelectiveTracker(
  skippedClassNames: string[],
  logger: (violation: string, source: string, key: string, value?: unknown) => void,
): <T>(value: T, source: string) => T;
```

## Dependencies { .dependencies }

### make-read-only-util 0.0.1 { .dependency }

Provides a factory function whose second argument is an array of class name strings. Objects
whose `constructor.name` is in that array are returned as-is without any property proxies
being installed, so mutations to them are never reported.

[@satisfied-by](make-read-only-util)
