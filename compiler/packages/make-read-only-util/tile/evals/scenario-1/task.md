# Immutability Violation Tracker

A development utility that monitors objects for unexpected mutations and logs detailed information when changes occur to objects that should remain immutable.

## Capabilities

### Tracks property mutations

- When a property of a tracked object is reassigned, the logger is called with violation type 'PROPERTY_MUTATED', the source identifier, property name, and new value [@test](../test/tracker.test.ts)
- When a nested object's property is mutated, the logger is called with the nested property name [@test](../test/tracker.test.ts)

### Detects property additions and deletions

- When a property is added to a tracked object between successive tracking calls, the logger is called with violation type 'PROPERTY_ADDED' [@test](../test/tracker.test.ts)
- When a property is deleted from a tracked object between successive tracking calls, the logger is called with violation type 'PROPERTY_DELETED' [@test](../test/tracker.test.ts)

### Supports class-based exclusions

- Objects whose constructor name is in the exclusion list are not tracked [@test](../test/tracker.test.ts)
- The tracker returns the original object unchanged for excluded classes [@test](../test/tracker.test.ts)

## Implementation

[@generates](./src/index.ts)

## API

```typescript { #api }
/**
 * Type representing the different kinds of immutability violations
 */
type ViolationType =
  | 'PROPERTY_MUTATED'
  | 'PROPERTY_DELETED'
  | 'PROPERTY_CHANGED'
  | 'PROPERTY_ADDED';

/**
 * Callback function that receives violation information
 */
type ViolationLogger = (
  violationType: ViolationType,
  sourceId: string,
  propertyKey: string,
  newValue?: any
) => void;

/**
 * Creates a tracking function configured with a logger and class exclusions
 *
 * @param logger - Callback invoked when violations are detected
 * @param excludedClasses - Array of constructor names to skip tracking
 * @returns A function that tracks objects for mutations
 */
export function createTracker(
  logger: ViolationLogger,
  excludedClasses: string[]
): <T>(obj: T, sourceId: string) => T;
```

## Dependencies { .dependencies }

### make-read-only-util { .dependency }

Provides utility to track mutations to objects marked as read-only.
