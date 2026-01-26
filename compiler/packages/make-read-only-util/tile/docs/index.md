# Make Read Only Util

Make Read Only Util is a specialized development utility for tracking mutations to objects marked as read-only in JavaScript/TypeScript applications. It provides a proxy-based mechanism that wraps objects with getters and setters to intercept property access and mutations, logging violations when read-only objects are mutated. This is particularly valuable for enforcing immutability contracts during development.

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

## Basic Usage

```typescript
import buildMakeReadOnly from "make-read-only-util";

// Define a violation logger
const logger = (violation, source, key, value) => {
  console.warn(`Violation: ${violation} in ${source}.${key}`, value);
};

// Build the makeReadOnly function with custom configuration
const makeReadOnly = buildMakeReadOnly(logger, []);

// Wrap an object to track mutations
const obj = { a: 1, b: { c: 2 } };
makeReadOnly(obj, "myComponent");

// Mutations are logged automatically
obj.a = 5; // Logs: FORGET_MUTATE_IMMUT

// Track property additions/deletions by calling makeReadOnly again
obj.d = 10;
makeReadOnly(obj, "myComponent"); // Logs: FORGET_ADD_PROP_IMMUT
```

## Capabilities

### Factory Function

Creates a customized `makeReadOnly` function with specific logging behavior and class exclusions.

```typescript { .api }
/**
 * Factory function that creates a makeReadOnly function with custom configuration
 * @param logger - Callback function invoked when violations are detected
 * @param skippedClasses - Array of class names to skip from read-only tracking
 * @returns A makeReadOnly function configured with the provided logger and exclusions
 */
function buildMakeReadOnly(
  logger: ROViolationLogger,
  skippedClasses: string[]
): <T>(val: T, source: string) => T;
```

**Parameters:**

- `logger`: Callback function that will be invoked whenever a mutation is detected on a read-only object. Receives violation type, source identifier, property key, and optional new value.
- `skippedClasses`: Array of class constructor names (strings) that should be excluded from read-only tracking. Objects whose `constructor.name` matches any entry in this array will be returned unchanged.

**Returns:**

A `makeReadOnly` function that can be used to wrap objects for mutation tracking.

**Usage Example:**

```typescript
import buildMakeReadOnly from "make-read-only-util";

// Create logger with custom behavior
const violations: Array<{ type: string; source: string; key: string }> = [];
const logger = (violation, source, key) => {
  violations.push({ type: violation, source, key });
};

// Skip tracking for Map and Set objects
const makeReadOnly = buildMakeReadOnly(logger, ["Map", "Set"]);

// Use the returned makeReadOnly function
const data = { items: [] };
makeReadOnly(data, "dataStore");
```

### Make Read Only Function

The function returned by `buildMakeReadOnly` wraps values to track mutations. This function maintains referential equality and handles primitive values, objects, and cyclic references.

```typescript { .api }
/**
 * Wraps a value to track mutations, returning the same value with proxy properties
 * @param val - The value to mark as read-only (primitives returned unchanged)
 * @param source - Source identifier for logging (helps identify where object was marked read-only)
 * @returns The same value (maintains referential equality for objects)
 */
function makeReadOnly<T>(val: T, source: string): T;
```

**Parameters:**

- `val`: The value to mark as read-only. Can be any type - primitives and null are returned unchanged, only objects are wrapped with proxies.
- `source`: A string identifier used in logging to identify where the object was marked as read-only. This helps trace violations back to specific components or modules.

**Returns:**

The same value passed in (maintains referential equality). For objects, properties are wrapped with getter/setter proxies to track mutations.

**Behavior:**

- **Primitives and null**: Returned unchanged without any wrapping
- **Objects**: Wrapped with proxy properties that intercept mutations
- **Skipped classes**: Objects whose `constructor.name` is in the `skippedClasses` array are returned unchanged
- **Referential equality**: Always returns the same object reference (no cloning)
- **Cyclic references**: Correctly handles objects with circular references
- **Property tracking**: Logs direct mutations immediately, but requires calling `makeReadOnly` again to detect added/deleted properties
- **Exclusions**: Does not track accessor properties (properties with get/set), and does not track the `current` property (used for React refs)

**Usage Examples:**

```typescript
import buildMakeReadOnly from "make-read-only-util";

const logger = (violation, source, key, value) => {
  console.log(`${violation}: ${source}.${key} = ${value}`);
};

const makeReadOnly = buildMakeReadOnly(logger, []);

// Primitives pass through unchanged
const num = makeReadOnly(5, "test");
const bool = makeReadOnly(true, "test");
const nil = makeReadOnly(null, "test");

// Objects maintain referential equality
const obj = { x: 1 };
const wrapped = makeReadOnly(obj, "test");
console.log(obj === wrapped); // true

// Direct mutations are logged immediately
const data = { count: 0 };
makeReadOnly(data, "counter");
data.count = 1; // Logs: FORGET_MUTATE_IMMUT: counter.count = 1

// Nested mutations are tracked when properties are accessed
const nested = { user: { name: "Alice" } };
makeReadOnly(nested, "state");
nested.user.name = "Bob"; // Logs: FORGET_MUTATE_IMMUT: state.name = Bob

// Aliased references maintain tracking
const original = { value: 42 };
const alias = original;
makeReadOnly(original, "shared");
alias.value = 99; // Logs: FORGET_MUTATE_IMMUT: shared.value = 99

// Property additions/deletions require calling makeReadOnly again
const mutable = { a: 1 };
makeReadOnly(mutable, "store");
mutable.b = 2; // Not logged yet
makeReadOnly(mutable, "store"); // Logs: FORGET_ADD_PROP_IMMUT: store.b

delete mutable.a; // Not logged yet
makeReadOnly(mutable, "store"); // Logs: FORGET_DELETE_PROP_IMMUT: store.a

// Cyclic references work correctly
const cyclic: any = { name: "root" };
cyclic.self = cyclic;
makeReadOnly(cyclic, "circular");
cyclic.self.name = "updated"; // Logs: FORGET_MUTATE_IMMUT
```

## Types

### Violation Type

Enumeration of violation types that can occur when mutating read-only objects.

```typescript { .api }
type ROViolationType =
  | "FORGET_MUTATE_IMMUT"      // Direct mutation of a read-only property
  | "FORGET_DELETE_PROP_IMMUT" // Deletion of a property from a read-only object
  | "FORGET_CHANGE_PROP_IMMUT" // Property changed (deleted and re-added)
  | "FORGET_ADD_PROP_IMMUT";   // New property added to a read-only object
```

**Values:**

- `"FORGET_MUTATE_IMMUT"`: Logged when a property value is changed on a read-only object
- `"FORGET_DELETE_PROP_IMMUT"`: Logged when a property is deleted from a read-only object (detected on subsequent `makeReadOnly` calls)
- `"FORGET_CHANGE_PROP_IMMUT"`: Logged when a property is deleted and re-added between `makeReadOnly` calls
- `"FORGET_ADD_PROP_IMMUT"`: Logged when a new property is added to a read-only object (detected on subsequent `makeReadOnly` calls)

### Violation Logger

Callback function type for logging violations detected on read-only objects.

```typescript { .api }
/**
 * Callback function type for logging violations
 * @param violation - The type of violation that occurred
 * @param source - Source identifier passed to makeReadOnly
 * @param key - Property key that was mutated
 * @param value - Optional new value (provided for mutation and add operations)
 */
type ROViolationLogger = (
  violation: ROViolationType,
  source: string,
  key: string,
  value?: any
) => void;
```

**Parameters:**

- `violation`: The type of violation that occurred (one of the `ROViolationType` values)
- `source`: The source identifier string that was passed to `makeReadOnly` when the object was marked read-only
- `key`: The property key (string) that was mutated, added, or deleted
- `value`: Optional parameter containing the new value. Present for `FORGET_MUTATE_IMMUT` and `FORGET_ADD_PROP_IMMUT` violations

**Usage Example:**

```typescript
import buildMakeReadOnly from "make-read-only-util";

// Collect violations for testing
const violations: Array<{
  type: string;
  location: string;
  property: string;
  newValue?: any;
}> = [];

const logger = (violation, source, key, value) => {
  violations.push({
    type: violation,
    location: source,
    property: key,
    newValue: value,
  });
};

const makeReadOnly = buildMakeReadOnly(logger, []);

// Use makeReadOnly and inspect violations
const obj = { x: 1 };
makeReadOnly(obj, "component.state");
obj.x = 2;

console.log(violations);
// [{
//   type: "FORGET_MUTATE_IMMUT",
//   location: "component.state",
//   property: "x",
//   newValue: 2
// }]
```

## Key Characteristics

- **Referential Equality**: Objects maintain referential equality after wrapping - the same object reference is returned
- **Lazy Tracking**: Nested object mutations are tracked lazily as properties are accessed through the proxy getters
- **Cyclic Reference Support**: Correctly handles objects with circular references without infinite loops
- **Property Addition/Deletion Tracking**: Requires calling `makeReadOnly` again on the same object to detect newly added or deleted properties
- **Class Filtering**: Allows skipping specific class types from tracking via the `skippedClasses` parameter
- **Non-invasive**: Works with existing objects without requiring deep cloning or modification of the object structure
- **Development Tool**: Designed for use in development and debugging contexts, particularly within the React compiler toolchain

## Limitations

The following limitations are inherent to the current proxy-based implementation:

1. **Property Addition/Deletion**: Newly added or deleted properties are only detected when `makeReadOnly` is called again on the same object. Single-call usage will not detect these changes.

2. **Aliased Indirect Mutations**: Mutations made via references obtained before the object was marked read-only are not tracked. For example:
   ```typescript
   const inner = { x: 0 };
   const outer = { inner };
   makeReadOnly(outer, "test");
   inner.x = 42; // Not tracked because 'inner' reference existed before makeReadOnly
   ```

3. **Accessor Properties**: Properties with accessor descriptors (get/set) are not tracked and are excluded from the proxy mechanism.

4. **React Ref Current Property**: The special property `current` (used for React refs) is explicitly excluded from tracking.

## Error Handling

This utility does not throw errors - it only logs violations through the provided logger callback. All operations complete successfully regardless of mutations detected. The logging behavior is entirely controlled by the `ROViolationLogger` callback you provide to `buildMakeReadOnly`.
