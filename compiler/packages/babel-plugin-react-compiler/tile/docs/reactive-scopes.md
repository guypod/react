# Reactive Scopes

Reactive scopes represent memoization boundaries in compiled React code. The compiler analyzes component and hook dependencies to determine optimal reactive scopes for automatic memoization.

## Capabilities

### Print Reactive Function

Print reactive function to string for debugging and inspection.

```typescript { .api }
/**
 * Print reactive function to string
 * @param fn - Reactive function to print
 * @returns Formatted string representation showing reactive scopes
 */
function printReactiveFunction(fn: ReactiveFunction): string;
```

**Usage Example:**

```typescript
import { run, printReactiveFunction } from "babel-plugin-react-compiler";

const pipeline = run(functionPath, config, "Component", "c", null, null, code);

for (const stage of pipeline) {
  if (stage.kind === "reactive") {
    console.log(`=== ${stage.name} ===`);
    console.log(printReactiveFunction(stage.value));
  }
}
```

## Types

### ReactiveFunction

Reactive function representation with scope information.

```typescript { .api }
/**
 * Reactive function with memoization scope information
 */
interface ReactiveFunction {
  /**
   * Source location
   */
  loc: SourceLocation;

  /**
   * Function identifier (null for anonymous functions)
   */
  id: string | null;

  /**
   * Function parameters
   */
  params: Array<Place | SpreadPattern>;

  /**
   * Is generator function
   */
  generator: boolean;

  /**
   * Is async function
   */
  async: boolean;

  /**
   * Function body with reactive scopes
   */
  body: ReactiveBlock;

  /**
   * Compilation environment
   */
  env: Environment;

  /**
   * Directive strings (e.g., "use strict", "use memo")
   */
  directives: Array<string>;
}
```

### ReactiveBlock

Block in reactive function representation.

```typescript { .api }
/**
 * Block in reactive function with scope information
 */
interface ReactiveBlock {
  /**
   * Block kind
   */
  kind: "block" | "value" | "loop" | "sequence" | "catch";

  /**
   * Block identifier
   */
  id: BlockId;

  /**
   * Instructions in the block
   */
  instructions: Array<ReactiveInstruction>;

  /**
   * Terminal node
   */
  terminal: ReactiveTerminal;

  /**
   * Reactive scope for this block (if any)
   */
  scope: ReactiveScope | null;
}
```

### ReactiveScope

Reactive scope with dependency tracking.

```typescript { .api }
/**
 * Reactive scope representing a memoization boundary
 */
interface ReactiveScope {
  /**
   * Unique scope identifier
   */
  id: ScopeId;

  /**
   * Instruction range covered by this scope
   */
  range: MutableRange;

  /**
   * Dependencies of this scope
   * Values that, when changed, invalidate the scope
   */
  dependencies: ReactiveScopeDependencies;

  /**
   * Declarations within this scope
   * Variables declared and their properties
   */
  declarations: Map<IdentifierId, ReactiveScopeDeclaration>;

  /**
   * Reassignments within the scope
   * Identifiers that are reassigned
   */
  reassignments: Set<Identifier>;

  /**
   * Early return value (if scope contains early return)
   */
  earlyReturnValue: Place | null;

  /**
   * Whether scope was pruned (optimized away)
   */
  pruned: boolean;

  /**
   * Parent scope (for nested scopes)
   */
  parent: ReactiveScope | null;

  /**
   * Child scopes (nested within this scope)
   */
  children: Set<ReactiveScope>;

  /**
   * Merged dependencies from child scopes
   */
  mergedDependencies: Set<IdentifierId>;
}
```

### ReactiveScopeDependencies

Dependencies for a reactive scope.

```typescript { .api }
/**
 * Dependencies that invalidate a reactive scope
 */
interface ReactiveScopeDependencies {
  /**
   * Direct dependencies
   * Identifiers directly read in the scope
   */
  dependencies: Set<IdentifierId>;

  /**
   * Conditional dependencies
   * Identifiers read conditionally
   */
  conditionalDependencies: Set<IdentifierId>;

  /**
   * Optional dependencies
   * Dependencies from optional chains
   */
  optionalDependencies: Set<IdentifierId>;

  /**
   * Access path dependencies
   * Property access chains (e.g., obj.a.b)
   */
  accessPaths: Map<IdentifierId, Array<string>>;
}
```

### ReactiveScopeDeclaration

Declaration within a reactive scope.

```typescript { .api }
/**
 * Variable declaration within a reactive scope
 */
interface ReactiveScopeDeclaration {
  /**
   * Identifier being declared
   */
  identifier: Identifier;

  /**
   * Scope where declared
   */
  scope: ReactiveScope;

  /**
   * Properties accessed on this identifier
   */
  accessedProperties: Set<string>;

  /**
   * Whether identifier is reassigned
   */
  reassigned: boolean;
}
```

### ScopeId

Unique scope identifier.

```typescript { .api }
/**
 * Unique reactive scope identifier
 */
type ScopeId = number & { readonly __brand: "ScopeId" };

/**
 * Create scope ID
 */
function makeScopeId(id: number): ScopeId;
```

### MutableRange

Range of instructions where a value is mutable.

```typescript { .api }
/**
 * Range of instructions where mutations are allowed
 */
interface MutableRange {
  /**
   * Start instruction ID (inclusive)
   */
  start: InstructionId;

  /**
   * End instruction ID (exclusive)
   */
  end: InstructionId;
}
```

## Reactive Scope Analysis Patterns

### Inspect Scope Dependencies

```typescript
import { type ReactiveFunction, type ReactiveScope } from "babel-plugin-react-compiler";

function analyzeScopeDependencies(scope: ReactiveScope) {
  console.log(`Scope ${scope.id}:`);

  console.log("  Direct dependencies:");
  for (const depId of scope.dependencies.dependencies) {
    console.log(`    - ${depId}`);
  }

  console.log("  Conditional dependencies:");
  for (const depId of scope.dependencies.conditionalDependencies) {
    console.log(`    - ${depId}`);
  }

  if (scope.dependencies.accessPaths.size > 0) {
    console.log("  Access paths:");
    for (const [id, path] of scope.dependencies.accessPaths) {
      console.log(`    - ${id}: ${path.join(".")}`);
    }
  }
}
```

### Find All Scopes

```typescript
import { type ReactiveFunction, type ReactiveScope } from "babel-plugin-react-compiler";

function findAllScopes(reactiveFn: ReactiveFunction): Array<ReactiveScope> {
  const scopes: Array<ReactiveScope> = [];

  function visitBlock(block: ReactiveBlock) {
    if (block.scope) {
      scopes.push(block.scope);
    }

    // Visit nested blocks based on terminal type
    // (implementation depends on terminal structure)
  }

  visitBlock(reactiveFn.body);
  return scopes;
}
```

### Scope Hierarchy

```typescript
import { type ReactiveScope } from "babel-plugin-react-compiler";

function printScopeHierarchy(scope: ReactiveScope, indent: number = 0) {
  const prefix = "  ".repeat(indent);

  console.log(`${prefix}Scope ${scope.id}`);
  console.log(`${prefix}  Range: [${scope.range.start}, ${scope.range.end}]`);
  console.log(`${prefix}  Dependencies: ${scope.dependencies.dependencies.size}`);
  console.log(`${prefix}  Declarations: ${scope.declarations.size}`);
  console.log(`${prefix}  Pruned: ${scope.pruned}`);

  if (scope.children.size > 0) {
    console.log(`${prefix}  Children:`);
    for (const child of scope.children) {
      printScopeHierarchy(child, indent + 2);
    }
  }
}
```

### Scope Metrics

```typescript
import { type ReactiveFunction } from "babel-plugin-react-compiler";

function calculateScopeMetrics(reactiveFn: ReactiveFunction) {
  const scopes = findAllScopes(reactiveFn);

  return {
    totalScopes: scopes.length,
    prunedScopes: scopes.filter((s) => s.pruned).length,
    avgDependencies:
      scopes.reduce((sum, s) => sum + s.dependencies.dependencies.size, 0) /
      scopes.length,
    maxDependencies: Math.max(
      ...scopes.map((s) => s.dependencies.dependencies.size)
    ),
    scopesWithConditionalDeps: scopes.filter(
      (s) => s.dependencies.conditionalDependencies.size > 0
    ).length,
  };
}
```

## Understanding Reactive Scopes

### What is a Reactive Scope?

A reactive scope represents a region of code that can be memoized as a unit. When any dependency changes, the entire scope is re-executed. The compiler automatically identifies these boundaries based on:

- Variable dependencies
- Property accesses
- Function calls
- Control flow

### Scope Dependencies

Dependencies determine when a scope needs to re-execute:

- **Direct dependencies**: Variables directly read in the scope
- **Conditional dependencies**: Variables read inside conditions
- **Optional dependencies**: Variables accessed via optional chaining
- **Access paths**: Specific properties accessed (e.g., `obj.a.b`)

### Scope Pruning

Some scopes are "pruned" (optimized away) when the compiler determines memoization provides no benefit:

- Scopes containing only hooks (hooks already memoize)
- Scopes with no observable outputs
- Scopes that would add overhead without benefit

### Nested Scopes

Scopes can be nested, creating a hierarchy:

```typescript
function Component({ data }) {
  // Outer scope
  const processed = expensiveProcess(data);

  return (
    <div>
      {/* Inner scope */}
      {processed.map(item => (
        <Item key={item.id} value={item.value} />
      ))}
    </div>
  );
}
```

The compiler creates separate scopes for:
1. The expensive processing
2. The map callback
3. The JSX rendering

## Debugging Reactive Scopes

### Print Readable Format

```typescript
import { run, printReactiveFunction } from "babel-plugin-react-compiler";

const pipeline = run(path, config, "Component", "c", null, "MyComponent.tsx", code);

for (const stage of pipeline) {
  if (stage.kind === "reactive") {
    console.log("=== Reactive Scopes ===");
    console.log(printReactiveFunction(stage.value));

    // Output shows scope boundaries, dependencies, and memoization decisions
  }
}
```

### Enable Memoization Comments

Add comments to generated code explaining memoization:

```typescript
const options = {
  environment: {
    enableMemoizationComments: true,
  },
};
```

Generated code will include comments like:

```javascript
function Component({ value }) {
  const $ = useMemoCache(2);

  // Scope 0: memoize doubled value
  let t0;
  if ($[0] !== value) {
    t0 = value * 2;
    $[0] = value;
    $[1] = t0;
  } else {
    t0 = $[1];
  }

  return t0;
}
```

### Analyze Scope Effectiveness

```typescript
import { type ReactiveFunction } from "babel-plugin-react-compiler";

function analyzeScopeEffectiveness(reactiveFn: ReactiveFunction) {
  const scopes = findAllScopes(reactiveFn);

  const effective = scopes.filter((s) => !s.pruned);
  const pruned = scopes.filter((s) => s.pruned);

  console.log(`Total scopes: ${scopes.length}`);
  console.log(`Effective scopes: ${effective.length}`);
  console.log(`Pruned scopes: ${pruned.length}`);
  console.log(`Effectiveness ratio: ${(effective.length / scopes.length * 100).toFixed(1)}%`);

  console.log("\nEffective scope details:");
  for (const scope of effective) {
    console.log(`  Scope ${scope.id}:`);
    console.log(`    Dependencies: ${scope.dependencies.dependencies.size}`);
    console.log(`    Declarations: ${scope.declarations.size}`);
  }
}
```
