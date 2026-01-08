# Cache Mutation Detector

Build a development tool that tracks mutations to cached objects in a memoization system. The tool should detect when cached values are modified, helping developers identify bugs where immutable cached data is being mutated.

## Requirements

### Core Functionality

Your implementation must:

1. Create a factory function that accepts a custom violation handler and a list of class names to exclude from tracking
2. The factory should return a function that wraps objects to track mutations
3. Track four types of violations:
   - Direct property mutations on tracked objects
   - Properties deleted from tracked objects
   - Properties replaced (deleted and re-added) on tracked objects
   - New properties added to tracked objects
4. Support deep tracking of nested objects - accessing nested properties should automatically track those nested values

### Violation Handler

The violation handler should receive:
- A violation type identifier
- A source identifier (to track where the object came from)
- The property name that was affected
- The new value (when applicable for mutations)

### Tracking Behavior

- Objects should maintain referential equality after wrapping (the function returns the same object instance)
- Primitive values (numbers, strings, booleans, null, undefined) should pass through unchanged
- Objects whose class name appears in the exclusion list should not be tracked
- Properties with existing getters/setters should be skipped
- Properties with a key named "current" should be skipped
- Only configurable, writeable properties should be tracked

### Detection Patterns

- Direct property mutations should be detected immediately when they occur
- Structural changes (deletions, additions, replacements) require calling the tracking function twice: once to establish a baseline, and again to detect the changes

### Implementation

[@generates](./src/tracker.ts)

### API

```typescript { #api }
export type ViolationType =
  | 'MUTATE'
  | 'DELETE'
  | 'CHANGE'
  | 'ADD';

export type ViolationHandler = (
  violation: ViolationType,
  source: string,
  key: string,
  value?: any,
) => void;

export function createTracker(
  handler: ViolationHandler,
  excludedClasses: string[],
): <T>(value: T, source: string) => T;
```

## Test Cases

- Calling the tracking function with a number returns that same number [@test](./src/tracker.test.ts)
- Setting a property on a tracked object calls the handler with violation type 'MUTATE' [@test](./src/tracker.test.ts)
- Setting a nested property calls the handler with the correct violation type [@test](./src/tracker.test.ts)
- Adding a new property and calling the tracking function again logs violation type 'ADD' [@test](./src/tracker.test.ts)
- Tracking an object of an excluded class returns it unchanged without any handler calls [@test](./src/tracker.test.ts)

## Dependencies { .dependencies }

### make-read-only-util { .dependency }

Provides runtime mutation tracking capabilities for read-only objects.
