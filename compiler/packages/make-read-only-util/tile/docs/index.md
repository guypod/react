# Make Read Only Util

Make Read Only Util is a TypeScript utility package for tracking and logging violations when code attempts to mutate objects that have been marked as read-only. It implements a proxy-based system that intercepts property modifications and logs different types of violations through a customizable logger function.

## Package Information

- **Package Name**: make-read-only-util
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: Part of React monorepo at `compiler/packages/make-read-only-util`
- **Repository**: https://github.com/facebook/react
- **License**: MIT

## Core Imports

```typescript
import buildMakeReadOnly from "make-read-only-util";
```

CommonJS:

```javascript
const buildMakeReadOnly = require("make-read-only-util");
```

## Basic Usage

```typescript
import buildMakeReadOnly from "make-read-only-util";

// Create a logger to handle violations
const logger = (violation, source, key, value) => {
  console.error(`[${source}] ${violation}: property '${key}' modified`, value);
};

// Build the makeReadOnly function with configuration
const makeReadOnly = buildMakeReadOnly(logger, []);

// Mark an object as read-only
const obj = { count: 0, name: "test" };
makeReadOnly(obj, "myComponent");

// Mutations are tracked and logged
obj.count = 5; // Logs: FORGET_MUTATE_IMMUT

// Nested mutations are also tracked
const nested = { data: { value: 42 } };
makeReadOnly(nested, "nestedExample");
nested.data.value = 100; // Logs: FORGET_MUTATE_IMMUT
```

## Capabilities

### Factory Function

Creates a configured `makeReadOnly` function that tracks mutations to objects.

```typescript { .api }
/**
 * Factory function that creates a makeReadOnly function configured with
 * a logger and list of classes to skip
 *
 * @param logger - Callback function that receives violation reports
 * @param skippedClasses - Array of class names to skip read-only enforcement
 * @returns A makeReadOnly function that accepts a value and source string
 */
function buildMakeReadOnly(
  logger: ROViolationLogger,
  skippedClasses: string[]
): <T>(val: T, source: string) => T;
```

The returned `makeReadOnly` function has the following signature:

```typescript { .api }
/**
 * Wraps an object to track mutations
 *
 * @param val - The value to make read-only (primitives pass through unchanged)
 * @param source - Source identifier used in logging violations
 * @returns The same value (by reference for objects, maintains referential equality)
 */
function makeReadOnly<T>(val: T, source: string): T;
```

**Behavior:**
- Primitives and null/undefined are returned unchanged
- Objects in the `skippedClasses` list are returned unchanged
- Other objects have their properties replaced with getter/setter proxies
- Maintains referential equality and handles cyclic references correctly
- Properties named 'current' are skipped (for React refs compatibility)
- Accessor properties (getters/setters) are not tracked

**Examples:**

```typescript
import buildMakeReadOnly from "make-read-only-util";

const violations: any[] = [];
const logger = (violation, source, key, value) => {
  violations.push({ violation, source, key, value });
};

const makeReadOnly = buildMakeReadOnly(logger, []);

// Basic usage
const obj = { x: 1, y: 2 };
makeReadOnly(obj, "example1");
obj.x = 10; // Violation logged: FORGET_MUTATE_IMMUT

// Nested object tracking
const parent = { child: { value: 42 } };
makeReadOnly(parent, "example2");
parent.child.value = 100; // Violation logged: FORGET_MUTATE_IMMUT

// Cyclic references
const circular: any = { self: null };
circular.self = circular;
makeReadOnly(circular, "example3"); // Works correctly

// Skipping specific classes
class SkippedClass {
  public value = 0;
}
const makeReadOnlyWithSkip = buildMakeReadOnly(logger, ["SkippedClass"]);
const skipped = new SkippedClass();
makeReadOnlyWithSkip(skipped, "example4");
skipped.value = 5; // No violation logged

// Detecting property additions/deletions
const mutable: any = { a: 1 };
makeReadOnly(mutable, "example5");
mutable.b = 2; // Not detected yet
makeReadOnly(mutable, "example5"); // Now detects: FORGET_ADD_PROP_IMMUT
delete mutable.a; // Not detected yet
makeReadOnly(mutable, "example5"); // Now detects: FORGET_DELETE_PROP_IMMUT
```

## Types

**Note**: The types below are not exported from the package but are provided here for reference. TypeScript users can infer these types from the `buildMakeReadOnly` function signature, or manually define them in their own code.

### ROViolationType

String literal union type representing different types of read-only violations.

```typescript { .api }
type ROViolationType =
  | 'FORGET_MUTATE_IMMUT'      // Mutation of an existing property
  | 'FORGET_DELETE_PROP_IMMUT'  // Deletion of a property
  | 'FORGET_CHANGE_PROP_IMMUT'  // Property was deleted and re-added
  | 'FORGET_ADD_PROP_IMMUT';    // New property was added
```

### ROViolationLogger

Function signature for the violation logger callback.

```typescript { .api }
type ROViolationLogger = (
  violation: ROViolationType,
  source: string,
  key: string,
  value?: any,
) => void;
```

**Parameters:**
- `violation: ROViolationType` - The type of violation that occurred
- `source: string` - The source identifier that was passed to makeReadOnly
- `key: string` - The property key that was accessed or mutated
- `value?: any` - Optional value involved in the violation (present for mutations, absent for deletions)

**Example:**

```typescript
const logger: ROViolationLogger = (violation, source, key, value) => {
  const timestamp = new Date().toISOString();
  const valueStr = value !== undefined ? `: ${JSON.stringify(value)}` : '';
  console.warn(`[${timestamp}] ${source}.${key} - ${violation}${valueStr}`);
};

const makeReadOnly = buildMakeReadOnly(logger, []);
const obj = { count: 0 };
makeReadOnly(obj, "Counter");
obj.count = 1; // Logs: [2026-01-23T...] Counter.count - FORGET_MUTATE_IMMUT: 1
```

## Violation Detection Behavior

### Tracked Violations

The following mutations are detected and logged:

1. **Direct property mutations** - Modifying a property value directly
   ```typescript
   obj.prop = newValue; // FORGET_MUTATE_IMMUT
   ```

2. **Transitive mutations** - Modifying nested object properties
   ```typescript
   obj.nested.prop = value; // FORGET_MUTATE_IMMUT
   ```

3. **Property additions** - Adding new properties (detected on subsequent makeReadOnly calls)
   ```typescript
   makeReadOnly(obj, "source");
   obj.newProp = value;
   makeReadOnly(obj, "source"); // FORGET_ADD_PROP_IMMUT
   ```

4. **Property deletions** - Deleting properties (detected on subsequent makeReadOnly calls)
   ```typescript
   makeReadOnly(obj, "source");
   delete obj.prop;
   makeReadOnly(obj, "source"); // FORGET_DELETE_PROP_IMMUT
   ```

5. **Property replacements** - Deleting and re-adding a property
   ```typescript
   makeReadOnly(obj, "source");
   delete obj.prop;
   obj.prop = newValue;
   makeReadOnly(obj, "source"); // FORGET_CHANGE_PROP_IMMUT
   ```

### Not Tracked

The following mutations are **not** detected:

1. **Property additions/deletions without subsequent makeReadOnly calls** - One-time call cannot detect these changes

2. **Mutations through pre-existing aliases** - References captured before makeReadOnly was called
   ```typescript
   const alias = obj.nested;
   makeReadOnly(obj, "source");
   alias.prop = value; // Not detected
   ```

3. **Accessor properties** - Properties with custom getters/setters are not tracked

4. **Properties named 'current'** - Special case for React refs compatibility

5. **Non-configurable or non-writable properties** - Properties that cannot be proxied

## Use Cases

### Development-Time Immutability Enforcement

Track violations during development to catch unintended mutations:

```typescript
const makeReadOnly = buildMakeReadOnly(
  (violation, source, key, value) => {
    if (process.env.NODE_ENV === 'development') {
      console.error(`Immutability violation in ${source}: ${key}`);
      throw new Error(`Attempted to modify read-only property: ${key}`);
    }
  },
  []
);
```

### React Compiler Integration

Track mutations in React components to enforce immutability constraints:

```typescript
const makeReadOnly = buildMakeReadOnly(
  (violation, source, key, value) => {
    console.warn(`Component ${source} mutated ${key} - may cause rendering issues`);
  },
  ['RefObject'] // Skip React ref objects
);

function MyComponent(props) {
  makeReadOnly(props, 'MyComponent.props');
  // Any prop mutations will be logged
}
```

### Debugging State Mutations

Identify where state is being unexpectedly modified:

```typescript
const mutations: Array<{ source: string; key: string; stack: string }> = [];

const makeReadOnly = buildMakeReadOnly(
  (violation, source, key, value) => {
    mutations.push({
      source,
      key,
      stack: new Error().stack || '',
    });
  },
  []
);

makeReadOnly(appState, 'AppState');
// Later, inspect mutations array to see what changed
```

## Performance Considerations

- **Memory efficient**: Uses WeakMap for caching, allowing garbage collection
- **Lazy proxying**: Properties are proxied on access, not eagerly
- **Overhead**: Adds getter/setter overhead to property access
- **Development only**: Intended for development and debugging, not production
- **Referential equality**: Maintains same object references, no cloning overhead
