# Immutability Violation Tracker

Build a utility that tracks and reports violations when objects marked as immutable are mutated during development.

## Problem Description

You need to implement a development-time monitoring system that detects when supposedly read-only objects are modified. The system should track various types of violations and provide detailed reports about where and how immutability contracts are broken.

Your implementation should:

1. Create a configurable tracking system via a factory function
2. Track objects and their properties to detect mutations
3. Categorize violations into different types
4. Provide violation statistics and reporting
5. Support filtering certain object types from tracking

## Requirements

### Factory Function

Implement a factory function that creates tracker instances with custom configuration:
- Accept a list of class names to exclude from tracking
- Return a tracking function that can be applied to objects

### Tracking Function

The tracking function should:
- Accept an object and a source identifier string
- Return the same object (maintain referential equality)
- Set up monitoring for property mutations
- Support calling multiple times on the same object to detect structural changes
- Handle primitive values, null, and undefined safely
- Track nested object mutations automatically

### Violation Detection

Detect and categorize four types of violations:
1. **Property Mutation**: When a property value is changed directly
2. **Property Addition**: When new properties are added to a previously tracked object
3. **Property Deletion**: When properties are removed from a tracked object
4. **Property Replacement**: When a property is deleted and then re-added

### Violation Reporting

Implement a reporting system that:
- Collects all violations that occur
- Provides a summary with counts for each violation type
- Includes details about each violation (type, source, property name, value if applicable)
- Can be queried at any time to get current statistics

### Special Handling

- Skip objects whose constructor name matches the exclusion list
- Skip properties with custom getters or setters
- Skip properties named 'current'
- Properly handle cyclic references without infinite loops

## API

```typescript { #api }
/**
 * Creates a tracker factory that monitors object mutations
 *
 * @param excludedClasses - Array of constructor names to skip tracking
 * @returns An object with trackObject function and reporting methods
 */
export function createMutationTracker(excludedClasses: string[]): {
  trackObject: <T>(obj: T, source: string) => T;
  getViolations: () => Array<{
    type: 'mutation' | 'addition' | 'deletion' | 'replacement';
    source: string;
    property: string;
    value?: any;
  }>;
  getViolationSummary: () => {
    mutation: number;
    addition: number;
    deletion: number;
    replacement: number;
  };
  reset: () => void;
};
```

## Test Cases

### Basic tracking [@test](./test/basic.test.ts)

```typescript
const tracker = createMutationTracker([]);
const obj = { count: 0 };

tracker.trackObject(obj, 'test1');
obj.count = 5;

const violations = tracker.getViolations();
// Should have 1 violation of type 'mutation'
```

### Nested object tracking [@test](./test/nested.test.ts)

```typescript
const tracker = createMutationTracker([]);
const obj = { data: { value: 10 } };

tracker.trackObject(obj, 'test2');
obj.data.value = 20;

const violations = tracker.getViolations();
// Should detect mutation on nested property 'value'
```

### Structural changes [@test](./test/structural.test.ts)

```typescript
const tracker = createMutationTracker([]);
const obj: any = { a: 1 };

tracker.trackObject(obj, 'test3');
obj.b = 2;
tracker.trackObject(obj, 'test3');

const summary = tracker.getViolationSummary();
// Should have 1 addition violation
```

### Class exclusion [@test](./test/exclusion.test.ts)

```typescript
class CustomClass {
  value = 0;
}

const tracker = createMutationTracker(['CustomClass']);
const obj = new CustomClass();

tracker.trackObject(obj, 'test4');
obj.value = 10;

const violations = tracker.getViolations();
// Should have 0 violations (CustomClass is excluded)
```

## Implementation

[@generates](./src/index.ts)

## Dependencies { .dependencies }

### make-read-only-util { .dependency }

Provides the underlying read-only object tracking mechanism with proxy-based property interception.

