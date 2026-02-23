# Read-Only Tracking with Referential Equality

A utility module that applies read-only mutation tracking to objects and confirms that the tracking function returns the exact same object reference that was passed in (referential equality).

## Capabilities

### Preserve object identity when applying read-only tracking

After applying read-only tracking to an object, the returned value must be strictly equal (`===`) to the input object. The tracking function modifies the object's property descriptors in place but does not create a new object or a wrapper.

- Applying the tracker to a plain object `{}` returns the same object (`===`) [@test](./tests/solution.test.ts)
- Applying the tracker to an object with nested object properties returns the top-level object (`===`) [@test](./tests/solution.test.ts)
- Applying the tracker to the same object twice returns the same object both times (`===`) [@test](./tests/solution.test.ts)

## Implementation

[@generates](./src/solution.ts)

## API

```typescript { #api }
/**
 * Applies read-only tracking to an object and returns the same object reference.
 * @param obj - The object to track
 * @param source - A string identifier for the tracking source
 * @returns The same object reference (strict equality preserved)
 */
export function trackObject<T extends object>(obj: T, source: string): T;
```

## Dependencies { .dependencies }

### make-read-only-util 0.0.1 { .dependency }

Provides a factory function returning a tracking function. The tracking function installs property
proxies on objects in place, preserving referential equality — the tracked object is the same
object that was passed in.

[@satisfied-by](make-read-only-util)
