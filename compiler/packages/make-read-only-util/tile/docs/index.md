# make-read-only-util

A runtime tracking system for detecting mutations to objects marked as "read-only", designed for React Compiler development and debugging. The library provides a proxy-based mechanism that monitors property access and modifications on designated objects, logging violations when mutations occur.

## Package Information

- **Package Name**: make-read-only-util
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: `npm install make-read-only-util`

## Core Imports

```typescript
import buildMakeReadOnly from "make-read-only-util";
```

For CommonJS:

```javascript
const buildMakeReadOnly = require("make-read-only-util");
```

### Type Imports

The package exports TypeScript type definitions:

```typescript
import buildMakeReadOnly, { type ROViolationType, type ROViolationLogger } from "make-read-only-util";
```

For TypeScript projects using CommonJS:

```typescript
import type { ROViolationType, ROViolationLogger } from "make-read-only-util";
const buildMakeReadOnly = require("make-read-only-util");
```

## Basic Usage

```typescript
import buildMakeReadOnly from "make-read-only-util";

// Create a logger to handle violations
const logger = (violation, source, key, value) => {
  console.error(`[${violation}] ${source}.${key} = ${value}`);
};

// Build a makeReadOnly function with logging configuration
const makeReadOnly = buildMakeReadOnly(logger, []);

// Mark an object as read-only
const obj = { a: 1, b: { c: 2 } };
makeReadOnly(obj, "myObject");

// Mutations are now tracked and logged
obj.a = 5; // Logs: [FORGET_MUTATE_IMMUT] myObject.a = 5
obj.b.c = 10; // Logs: [FORGET_MUTATE_IMMUT] myObject.c = 10
```

## Capabilities

### Build makeReadOnly Function

Factory function that creates a makeReadOnly function with configured logging and class filtering.

```typescript { .api }
/**
 * Creates a makeReadOnly function with violation logging and class filtering
 * @param logger - Callback function invoked when violations are detected
 * @param skippedClasses - Array of class names to exclude from tracking
 * @returns A makeReadOnly function for tracking object mutations
 */
function buildMakeReadOnly(
  logger: ROViolationLogger,
  skippedClasses: string[]
): <T>(val: T, source: string) => T;
```

**Usage Example:**

```typescript
import buildMakeReadOnly from "make-read-only-util";

const violations: Array<{type: string; source: string; key: string}> = [];

const logger = (violation, source, key, value) => {
  violations.push({ type: violation, source, key });
};

// Skip tracking for certain class instances
const makeReadOnly = buildMakeReadOnly(logger, ["WeakMap", "WeakSet"]);

const obj = { count: 0 };
makeReadOnly(obj, "counter");
obj.count = 1; // Violation logged
```

### makeReadOnly Function

The function returned by `buildMakeReadOnly` that marks objects as read-only and tracks mutations.

```typescript { .api }
/**
 * Marks a value as read-only and tracks mutations via proxy mechanism
 * @param val - The value to mark as read-only (any type)
 * @param source - Identifier string used in logging to track the origin
 * @returns The same value with referential equality maintained
 */
function makeReadOnly<T>(val: T, source: string): T;
```

**Behavior:**

- **Primitives and null**: Returned unchanged without tracking
- **Filtered classes**: Objects matching `skippedClasses` are returned unchanged
- **Regular objects**: Properties replaced with getter/setter proxies
- **Nested objects**: Recursively tracked when accessed (lazy tracking)
- **Multiple calls**: Detects property additions and deletions between calls
- **Referential equality**: The same object reference is returned

**Usage Example:**

```typescript
// Track immediate mutations
const obj = { x: 1 };
makeReadOnly(obj, "test");
obj.x = 2; // Logs: FORGET_MUTATE_IMMUT

// Track nested mutations
const nested = { outer: { inner: 5 } };
makeReadOnly(nested, "test");
nested.outer.inner = 10; // Logs: FORGET_MUTATE_IMMUT

// Track property additions/deletions
const dynamic: any = {};
makeReadOnly(dynamic, "test");
dynamic.newProp = "value";
makeReadOnly(dynamic, "test"); // Logs: FORGET_ADD_PROP_IMMUT

const withProp: any = { prop: 1 };
makeReadOnly(withProp, "test");
delete withProp.prop;
makeReadOnly(withProp, "test"); // Logs: FORGET_DELETE_PROP_IMMUT
```

**Limitations:**

- Does not track properties with custom getter/setter accessors
- Does not track properties named `'current'` (React ref optimization)
- Properties deleted/added without subsequent `makeReadOnly` call are not detected
- Aliased mutations occurring before the first `makeReadOnly` call are not tracked

### Violation Logger Callback

Callback function invoked when a read-only violation is detected.

```typescript { .api }
/**
 * Callback function signature for logging read-only violations
 * @param violation - Type of violation that occurred
 * @param source - Source identifier passed to makeReadOnly
 * @param key - Property key that was modified
 * @param value - New value assigned (undefined for deletions)
 */
type ROViolationLogger = (
  violation: ROViolationType,
  source: string,
  key: string,
  value?: any
) => void;
```

**Usage Example:**

```typescript
import buildMakeReadOnly from "make-read-only-util";

// Custom logger with formatted output
const logger: ROViolationLogger = (violation, source, key, value) => {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${violation}`);
  console.log(`  Source: ${source}`);
  console.log(`  Key: ${key}`);
  if (value !== undefined) {
    console.log(`  Value: ${JSON.stringify(value)}`);
  }
};

const makeReadOnly = buildMakeReadOnly(logger, []);
```

## Types

### ROViolationType

Enumeration of possible violation types detected by the tracking system.

```typescript { .api }
type ROViolationType =
  | "FORGET_MUTATE_IMMUT"    // Direct mutation of a property value
  | "FORGET_DELETE_PROP_IMMUT"  // Property deletion
  | "FORGET_CHANGE_PROP_IMMUT"  // Property changed (deleted and re-added)
  | "FORGET_ADD_PROP_IMMUT";    // New property added
```

**Violation Types:**

- **`FORGET_MUTATE_IMMUT`**: A property value was directly mutated
- **`FORGET_DELETE_PROP_IMMUT`**: A property was deleted from the object
- **`FORGET_CHANGE_PROP_IMMUT`**: A property was deleted and then re-added (detected between `makeReadOnly` calls)
- **`FORGET_ADD_PROP_IMMUT`**: A new property was added (detected between `makeReadOnly` calls)

### ROViolationLogger

Type definition for the violation logger callback function.

```typescript { .api }
type ROViolationLogger = (
  violation: ROViolationType,
  source: string,
  key: string,
  value?: any
) => void;
```

## Advanced Usage

### Class Filtering

Exclude specific class instances from tracking to avoid overhead or conflicts:

```typescript
import buildMakeReadOnly from "make-read-only-util";

class InternalBuffer {
  data: Uint8Array;
  constructor() {
    this.data = new Uint8Array(1024);
  }
}

// Skip tracking for InternalBuffer instances
const makeReadOnly = buildMakeReadOnly(
  (violation, source, key) => console.log(`${violation}: ${source}.${key}`),
  ["InternalBuffer", "WeakMap"]
);

const obj = {
  buffer: new InternalBuffer(),
  config: { setting: true }
};

makeReadOnly(obj, "app");
// obj.buffer mutations are not tracked
// obj.config mutations ARE tracked
```

### Integration with Testing Frameworks

Use in test suites to validate immutability constraints:

```typescript
import buildMakeReadOnly from "make-read-only-util";

describe("Component Tests", () => {
  let violations: Array<{type: string; source: string; key: string}>;
  let makeReadOnly: <T>(val: T, source: string) => T;

  beforeEach(() => {
    violations = [];
    const logger = (violation, source, key) => {
      violations.push({ type: violation, source, key });
    };
    makeReadOnly = buildMakeReadOnly(logger, []);
  });

  it("should not mutate props", () => {
    const props = { value: 42 };
    makeReadOnly(props, "props");

    // Component code that should not mutate props
    renderComponent(props);

    expect(violations).toHaveLength(0);
  });
});
```

### Multi-Call Tracking Pattern

Track property additions and deletions by calling `makeReadOnly` multiple times:

```typescript
import buildMakeReadOnly from "make-read-only-util";

const logger = (violation, source, key) => {
  console.log(`${violation} detected on ${source}.${key}`);
};

const makeReadOnly = buildMakeReadOnly(logger, []);

const state: any = { initialized: true };

// First call establishes baseline
makeReadOnly(state, "appState");

// Modify state
state.userCount = 100; // Not logged yet

// Second call detects changes
makeReadOnly(state, "appState"); // Logs: FORGET_ADD_PROP_IMMUT

// More modifications
delete state.initialized; // Not logged yet

// Third call detects deletion
makeReadOnly(state, "appState"); // Logs: FORGET_DELETE_PROP_IMMUT
```

### React Compiler Integration

The package is designed for React Compiler development to ensure generated code respects immutability:

```typescript
import buildMakeReadOnly from "make-read-only-util";

// In React Compiler test suite
const logger = (violation, source, key, value) => {
  throw new Error(
    `Immutability violation in compiled code: ${violation} at ${source}.${key}`
  );
};

const makeReadOnly = buildMakeReadOnly(logger, []);

// Wrap compiler-generated function inputs
function testCompiledComponent(props: any) {
  makeReadOnly(props, "componentProps");
  return compiledComponentCode(props);
}
```
