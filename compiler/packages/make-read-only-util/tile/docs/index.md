# make-read-only-util

`make-read-only-util` is a TypeScript utility for tracking mutations to objects designated as read-only. It installs property descriptor proxies on objects so that any subsequent write, deletion, addition, or modification of a property triggers a provided violation logger callback. Used by the React Compiler project to detect illegal mutations to values the compiler assumes are immutable.

## Package Information

- **Package Name**: make-read-only-util
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: `npm install make-read-only-util`

## Core Imports

```typescript
import buildMakeReadOnly from 'make-read-only-util';
```

CommonJS:

```javascript
const buildMakeReadOnly = require('make-read-only-util');
```

## Basic Usage

```typescript
import buildMakeReadOnly from 'make-read-only-util';

// Create a violation logger
const logger = (violation, source, key, value) => {
  console.warn(`Mutation detected [${violation}] in ${source}: key="${key}"`, value);
};

// Build a makeReadOnly function (skipping no classes)
const makeReadOnly = buildMakeReadOnly(logger, []);

// Mark an object as read-only
const obj = { x: 1, y: 2 };
makeReadOnly(obj, 'MyComponent');

// Any subsequent mutation triggers the logger
obj.x = 99; // logs: FORGET_MUTATE_IMMUT, 'MyComponent', 'x', 99
```

## Capabilities

### Building the makeReadOnly Function

`buildMakeReadOnly` is the package's sole export. It is a factory function that creates a `makeReadOnly` function bound to a specific violation logger and a set of classes to skip.

```typescript { .api }
/**
 * Factory that creates a makeReadOnly function bound to the given logger and skipped classes.
 *
 * @param logger - Callback invoked on any detected mutation of a tracked object.
 * @param skippedClasses - Class constructor names to skip (objects of these classes are
 *   passed through without installing proxies).
 * @returns A makeReadOnly function: <T>(val: T, source: string) => T
 */
function buildMakeReadOnly(
  logger: ROViolationLogger,
  skippedClasses: string[]
): <T>(val: T, source: string) => T;
```

### Applying Read-Only Tracking

The function returned by `buildMakeReadOnly` installs property descriptor proxies on an object's writable, configurable own properties. Returns the input value unchanged (preserves referential equality).

```typescript { .api }
/**
 * Marks an object as read-only by installing setter-intercepting property descriptors.
 * Primitives and null are returned as-is. Objects in skippedClasses are passed through.
 * Tracking is transitive: when a proxied property is accessed, makeReadOnly is also
 * applied to the returned value.
 *
 * @param val - Value to mark as read-only. Primitives and null are returned unchanged.
 * @param source - Identifier string used when reporting violations (e.g. component name).
 * @returns The same input value (referential equality is preserved for objects).
 */
function makeReadOnly<T>(val: T, source: string): T;
```

**Behavior:**
- Primitives (`number`, `boolean`, `string`, etc.) and `null` are returned as-is with no tracking.
- Objects whose `constructor.name` is in `skippedClasses` are returned without any proxies.
- Accessor properties (those with existing `get`/`set` descriptors) are skipped and not tracked.
- Properties named `current` are skipped (to avoid intercepting React `ref` objects).
- A `WeakMap`-based cache prevents infinite loops on cyclic object graphs.
- **Single call limitation**: If `makeReadOnly` is called only once on an object, subsequently added or deleted properties are **not** detected. To detect additions and deletions, call `makeReadOnly` again on the same object after mutations may have occurred.

**Usage Examples:**

```typescript
import buildMakeReadOnly from 'make-read-only-util';

const violations: string[] = [];
const logger = (violation, source, key, value?) => {
  violations.push(`${violation}:${source}:${key}`);
};

const makeReadOnly = buildMakeReadOnly(logger, []);

// --- Direct mutation detection ---
const obj = { a: 0 };
makeReadOnly(obj, 'source1');
obj.a = 42; // triggers: FORGET_MUTATE_IMMUT, 'source1', 'a', 42

// --- Transitive mutation detection ---
const nested = { inner: { x: 0 } };
makeReadOnly(nested, 'source2');
nested.inner.x = 99; // triggers: FORGET_MUTATE_IMMUT, 'source2', 'x', 99

// --- Detecting added properties (requires second call) ---
const o: any = {};
makeReadOnly(o, 'source3');
o.newProp = 'hello';
makeReadOnly(o, 'source3'); // triggers: FORGET_ADD_PROP_IMMUT, 'source3', 'newProp'

// --- Detecting deleted properties (requires second call) ---
const o2: any = { a: 1 };
makeReadOnly(o2, 'source4');
delete o2.a;
makeReadOnly(o2, 'source4'); // triggers: FORGET_DELETE_PROP_IMMUT, 'source4', 'a'

// --- Skipping specific classes ---
const makeReadOnlySkipMap = buildMakeReadOnly(logger, ['Map', 'Set']);
const m = new Map();
makeReadOnlySkipMap(m, 'source5'); // Map is skipped, no proxies installed

// --- Cyclic objects are handled safely ---
const cyclic: any = {};
cyclic.self = cyclic;
makeReadOnly(cyclic, 'source6'); // no infinite loop
```

## Types

```typescript { .api }
/**
 * The type of mutation violation detected on a read-only object.
 *
 * - FORGET_MUTATE_IMMUT:      A tracked property's value was set (immediate detection).
 * - FORGET_DELETE_PROP_IMMUT: A tracked property was deleted (detected on next makeReadOnly call).
 * - FORGET_CHANGE_PROP_IMMUT: A tracked property was deleted then re-added (detected on next call).
 * - FORGET_ADD_PROP_IMMUT:    A new property was added to a previously-tracked object (detected on next call).
 */
type ROViolationType =
  | 'FORGET_MUTATE_IMMUT'
  | 'FORGET_DELETE_PROP_IMMUT'
  | 'FORGET_CHANGE_PROP_IMMUT'
  | 'FORGET_ADD_PROP_IMMUT';

/**
 * Callback invoked whenever a mutation is detected on a read-only object.
 *
 * @param violation - The category of mutation detected.
 * @param source    - The source identifier provided to makeReadOnly.
 * @param key       - The property key that was mutated/added/deleted.
 * @param value     - The new value being set (only provided for FORGET_MUTATE_IMMUT violations).
 */
type ROViolationLogger = (
  violation: ROViolationType,
  source: string,
  key: string,
  value?: any,
) => void;
```

## Violation Types Reference

| Violation | Trigger Condition | `value` parameter |
|-----------|-------------------|-------------------|
| `FORGET_MUTATE_IMMUT` | A proxied property is directly assigned a new value | The new value being set |
| `FORGET_DELETE_PROP_IMMUT` | A proxied property is deleted; detected when `makeReadOnly` is called again on the same object | Not provided |
| `FORGET_CHANGE_PROP_IMMUT` | A proxied property is deleted and then re-added between calls to `makeReadOnly` | Not provided |
| `FORGET_ADD_PROP_IMMUT` | A new property is added to a previously tracked object; detected when `makeReadOnly` is called again | Not provided |

## Limitations

- **No proxy for accessor properties**: Properties that already have `get`/`set` descriptors are not wrapped and will not trigger violation logging.
- **No proxy for `current` property**: Properties named `current` are explicitly skipped (avoids wrapping React ref objects).
- **Single-call limitation for add/delete**: Property additions and deletions can only be detected if `makeReadOnly` is called again on the object after the mutation. A single call does not set up Proxy traps for structural changes.
- **Non-configurable properties skipped**: Only writable and configurable properties receive proxy descriptors.
- **Aliased direct references not tracked**: Tracking is lazy — inner objects are only proxied when accessed through their parent's getter. If you hold a direct reference to a nested object and mutate it without going through the parent proxy, the mutation will not be detected.

  ```typescript
  const inner = { x: 0 };
  const outer = { a: inner };
  makeReadOnly(outer, 'source');

  // NOT tracked — mutating inner directly bypasses the lazy proxy
  inner.x = 42;

  // IS tracked — accessing outer.a triggers the getter, which proxies inner
  outer.a.x = 42;
  ```
