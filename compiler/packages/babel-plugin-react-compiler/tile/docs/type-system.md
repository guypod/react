# Type System

The React Compiler includes a type system for tracking effects and value kinds to make optimization decisions. This system determines how values can be used and what side effects operations may have.

## Capabilities

### Effect Enum

Effect types for tracking how operations affect their arguments.

```typescript { .api }
/**
 * Effect types for tracking mutations and side effects
 */
enum Effect {
  /**
   * Unknown effect - effect cannot be determined
   * Used when effect analysis is incomplete
   */
  Unknown = "<unknown>",

  /**
   * Freeze effect - value is made immutable
   * Prevents future mutations
   */
  Freeze = "freeze",

  /**
   * Read effect - value is read but not modified
   * Safe to reorder with other reads
   */
  Read = "read",

  /**
   * Capture effect - value is captured (e.g., in closure)
   * Value reference is stored but not immediately used
   */
  Capture = "capture",

  /**
   * Conditionally mutate - value may be mutated
   * Mutation happens conditionally (e.g., in if statement)
   */
  ConditionallyMutate = "mutate?",

  /**
   * Mutate effect - value is modified
   * Cannot reorder with other operations on same value
   */
  Mutate = "mutate",

  /**
   * Store effect - value is stored to a location
   * Represents assignment operations
   */
  Store = "store",
}
```

**Usage Example:**

```typescript
import { Effect } from "babel-plugin-react-compiler";

// Define custom hook with read effect
const customHook = {
  effectKind: Effect.Read,
  valueKind: ValueKind.Mutable,
  noAlias: false,
  transitiveMixedData: false,
};
```

### ValueKind Enum

Value kinds for optimization decisions.

```typescript { .api }
/**
 * Value kinds for optimization decisions
 */
enum ValueKind {
  /**
   * Maybe frozen value - may be frozen or mutable
   * Conservatively treated as potentially mutable
   */
  MaybeFrozen = "maybefrozen",

  /**
   * Frozen value - immutable object/array
   * Cannot be modified, safe to share across scopes
   */
  Frozen = "frozen",

  /**
   * Primitive value - immutable primitive type
   * Numbers, strings, booleans, null, undefined, symbols
   * Safe to freely copy and compare
   */
  Primitive = "primitive",

  /**
   * Global value - global variable or import
   * Special handling for global scope access
   */
  Global = "global",

  /**
   * Mutable value - can be modified
   * Requires careful dependency tracking
   */
  Mutable = "mutable",

  /**
   * Context value - React context value
   * Special handling for context propagation
   */
  Context = "context",
}
```

**Usage Example:**

```typescript
import { ValueKind } from "babel-plugin-react-compiler";

// Check if a value can be safely memoized
function canMemoize(valueKind: ValueKind): boolean {
  return (
    valueKind === ValueKind.Primitive ||
    valueKind === ValueKind.Frozen
  );
}
```

## Types

### Type

Union of all type system types.

```typescript { .api }
/**
 * Type system type
 */
type Type = BuiltInType | PhiType | TypeVar | PolyType | PropType | ObjectMethod;

/**
 * Built-in types
 */
type BuiltInType = PrimitiveType | FunctionType | ObjectType;
```

### PrimitiveType

Primitive type representation.

```typescript { .api }
/**
 * Primitive type (numbers, strings, booleans, null, undefined)
 */
interface PrimitiveType {
  kind: "Primitive";
}
```

### FunctionType

Function type representation.

```typescript { .api }
/**
 * Function type
 */
interface FunctionType {
  /**
   * Type kind
   */
  kind: "Function";

  /**
   * Shape ID for this function type (null if unknown)
   */
  shapeId: string | null;

  /**
   * Return type
   */
  return: Type;
}
```

### ObjectType

Object type representation.

```typescript { .api }
/**
 * Object type
 */
interface ObjectType {
  kind: "Object";

  /**
   * Shape ID for this object type (null if unknown)
   */
  shapeId: string | null;
}
```

### PhiType

Phi type for SSA form (union of multiple types).

```typescript { .api }
/**
 * Phi type - represents union of multiple possible types
 * Used in SSA form for variables with different types in different branches
 */
interface PhiType {
  kind: "Phi";

  /**
   * Possible types
   */
  types: Set<Type>;
}
```

### TypeVar

Type variable for generic types.

```typescript { .api }
/**
 * Type variable for generics
 */
interface TypeVar {
  kind: "TypeVar";

  /**
   * Type variable name
   */
  name: string;
}
```

### PolyType

Polymorphic type (generic type with parameters).

```typescript { .api }
/**
 * Polymorphic type (generic)
 */
interface PolyType {
  kind: "Poly";

  /**
   * Type parameters
   */
  params: Array<TypeVar>;

  /**
   * Body type
   */
  body: Type;
}
```

## Type Checking Functions

Utility functions for type checking.

```typescript { .api }
/**
 * Check if type is object type
 */
function isObjectType(type: Type): type is ObjectType;

/**
 * Check if type is primitive type
 */
function isPrimitiveType(type: Type): type is PrimitiveType;

/**
 * Check if type is function type
 */
function isFunctionType(type: Type): type is FunctionType;

/**
 * Check if type is array type
 * @param env - Environment with type information
 * @param type - Type to check
 */
function isArrayType(env: Environment, type: Type): boolean;

/**
 * Check if type is ref value type (result of .current)
 * @param env - Environment with type information
 * @param type - Type to check
 */
function isRefValueType(env: Environment, type: Type): boolean;

/**
 * Check if type is useRef hook type
 * @param env - Environment with type information
 * @param type - Type to check
 */
function isUseRefType(env: Environment, type: Type): boolean;

/**
 * Check if type is useState hook type
 * @param env - Environment with type information
 * @param type - Type to check
 */
function isUseStateType(env: Environment, type: Type): boolean;
```

**Usage Example:**

```typescript
import {
  isArrayType,
  isPrimitiveType,
  isUseRefType,
  type Type,
  type Environment,
} from "babel-plugin-react-compiler";

function analyzeType(env: Environment, type: Type) {
  if (isPrimitiveType(type)) {
    console.log("Primitive type - safe to memoize");
  } else if (isArrayType(env, type)) {
    console.log("Array type - needs dependency tracking");
  } else if (isUseRefType(env, type)) {
    console.log("Ref type - special handling required");
  }
}
```

## Object Shape Registry

The compiler maintains a registry of object shapes for built-in types and custom types.

### Built-in Shape IDs

```typescript { .api }
/**
 * Built-in object shape identifiers
 */
const BuiltInArrayId: string; // Array shape
const BuiltInObjectId: string; // Object shape
const BuiltInFunctionId: string; // Function shape
const BuiltInPropsId: string; // Component props shape
const BuiltInJsxId: string; // JSX element shape
const BuiltInUseStateId: string; // useState hook shape
const BuiltInSetStateId: string; // setState function shape
const BuiltInUseRefId: string; // useRef hook shape
const BuiltInRefValueId: string; // Ref value shape
const BuiltInUseEffectId: string; // useEffect hook shape
const BuiltInUseLayoutEffectId: string; // useLayoutEffect hook shape
const BuiltInUseMemoId: string; // useMemo hook shape
const BuiltInUseCallbackId: string; // useCallback hook shape
const BuiltInUseContextId: string; // useContext hook shape
```

### Shape Registry Functions

```typescript { .api }
/**
 * Add function to shape registry
 * @param registry - Shape registry
 * @param properties - Function properties
 * @param fn - Function type
 * @param id - Optional shape ID (generated if not provided)
 * @returns Shape ID
 */
function addFunction(
  registry: ShapeRegistry,
  properties: Map<string, ShapeProperty>,
  fn: FunctionType,
  id?: string
): string;

/**
 * Add hook to shape registry
 * @param registry - Shape registry
 * @param fn - Hook configuration
 * @param id - Optional shape ID (generated if not provided)
 * @returns Shape ID
 */
function addHook(
  registry: ShapeRegistry,
  fn: Hook,
  id?: string
): string;

/**
 * Add object to shape registry
 * @param registry - Shape registry
 * @param id - Shape ID
 * @param properties - Object properties
 */
function addObject(
  registry: ShapeRegistry,
  id: string,
  properties: Map<string, ShapeProperty>
): void;
```

### ShapeProperty

Property in an object shape.

```typescript { .api }
/**
 * Property in an object shape
 */
interface ShapeProperty {
  /**
   * Property type
   */
  type: Type;

  /**
   * Is property optional
   */
  optional: boolean;

  /**
   * Is property readonly
   */
  readonly: boolean;
}
```

## Effect and Value Kind Combinations

Different combinations of effects and value kinds have different optimization implications:

### Read + Primitive
- Safe to freely copy
- No dependency tracking needed
- Can inline across scopes

### Read + Mutable
- Requires dependency tracking
- Cannot assume value unchanged
- Must track for memoization

### Read + Frozen
- Safe to share across scopes
- No mutation possible
- Optimal for memoization

### Mutate + Mutable
- Invalidates dependent scopes
- Cannot reorder with other operations
- Critical for dependency analysis

### Capture + Mutable
- Value captured in closure
- Must track for escape analysis
- Affects scope boundaries

## Type-based Optimizations

### TypeScript/Flow Type Annotations

When `enableUseTypeAnnotations` is enabled:

```typescript
const config = {
  enableUseTypeAnnotations: true,
};
```

The compiler uses TypeScript/Flow type annotations to make better optimization decisions:

```typescript
// Compiler knows `count` is always a number (primitive)
function Component({ count }: { count: number }) {
  const doubled = count * 2; // Can optimize as primitive operation
  return <div>{doubled}</div>;
}

// Compiler knows `items` is readonly array
function Component({ items }: { readonly items: readonly Item[] }) {
  // Can safely memoize without mutation concerns
  const processed = items.map(process);
  return <List items={processed} />;
}
```

### Custom Type Definitions

For React Native Reanimated:

```typescript
const config = {
  enableCustomTypeDefinitionForReanimated: true,
};
```

This enables special type handling for Reanimated's worklet functions and shared values.

## Practical Examples

### Analyze Value Mutability

```typescript
import { ValueKind } from "babel-plugin-react-compiler";

function shouldTrackDependency(valueKind: ValueKind): boolean {
  switch (valueKind) {
    case ValueKind.Primitive:
      return false; // Primitives don't need tracking

    case ValueKind.Frozen:
      return false; // Frozen objects are immutable

    case ValueKind.Mutable:
      return true; // Must track mutable values

    case ValueKind.Context:
      return true; // Context values need tracking
  }
}
```

### Effect-based Reordering

```typescript
import { Effect } from "babel-plugin-react-compiler";

function canReorder(effect1: Effect, effect2: Effect): boolean {
  // Read effects can be reordered with other reads
  if (effect1 === Effect.Read && effect2 === Effect.Read) {
    return true;
  }

  // Mutations cannot be reordered
  if (
    effect1 === Effect.Mutate ||
    effect2 === Effect.Mutate ||
    effect1 === Effect.ConditionallyMutate ||
    effect2 === Effect.ConditionallyMutate
  ) {
    return false;
  }

  // Store operations cannot be reordered
  if (effect1 === Effect.Store || effect2 === Effect.Store) {
    return false;
  }

  return true;
}
```

### Custom Hook Type Definition

```typescript
import { Effect, ValueKind, type Hook } from "babel-plugin-react-compiler";

// Define a custom data-fetching hook
const useDataFetch: Hook = {
  effectKind: Effect.Read, // Reads from external source
  valueKind: ValueKind.Mutable, // Returns mutable data
  noAlias: false, // Return value can be aliased
  transitiveMixedData: true, // Returns JSON data
};

// Define a custom immutable state hook
const useImmutableState: Hook = {
  effectKind: Effect.Read,
  valueKind: ValueKind.Frozen, // Returns immutable value
  noAlias: true, // No aliasing
  transitiveMixedData: false,
};

// Use in configuration
const config = {
  customHooks: new Map([
    ["useDataFetch", useDataFetch],
    ["useImmutableState", useImmutableState],
  ]),
};
```
