# HIR (High-level Intermediate Representation)

The High-level Intermediate Representation (HIR) is the compiler's internal control flow graph representation used for analysis and optimization. HIR provides a structured view of React components and hooks at a higher level of abstraction than AST.

## Capabilities

### Print HIR

Print HIR function to string for debugging and inspection.

```typescript { .api }
/**
 * Print HIR function to string
 * @param fn - HIR function to print
 * @returns Formatted string representation of HIR
 */
function printHIR(fn: HIRFunction): string;
```

**Usage Example:**

```typescript
import { run, printHIR } from "babel-plugin-react-compiler";

const pipeline = run(functionPath, config, "Component", "c", null, null, code);

for (const stage of pipeline) {
  if (stage.kind === "hir") {
    console.log(`=== ${stage.name} ===`);
    console.log(printHIR(stage.value));
  }
}
```

## Types

### HIRFunction

High-level intermediate representation of a function.

```typescript { .api }
/**
 * HIR function representation with control flow graph
 */
interface HIRFunction {
  /**
   * Source location of the function
   */
  loc: SourceLocation;

  /**
   * Function identifier (null for anonymous functions)
   */
  id: string | null;

  /**
   * React function type classification
   * - 'Component': React component
   * - 'Hook': React hook
   * - 'Other': Regular function
   */
  fnType: ReactFunctionType;

  /**
   * Compilation environment with type info and config
   */
  env: Environment;

  /**
   * Function parameters
   */
  params: Array<Place | SpreadPattern>;

  /**
   * Return type annotation (TypeScript/Flow)
   */
  returnType: t.FlowType | t.TSType | null;

  /**
   * Context variables captured from outer scope
   */
  context: Array<Place>;

  /**
   * Function effects (side effects analysis)
   */
  effects: Array<FunctionEffect> | null;

  /**
   * Function body as control flow graph
   */
  body: HIR;

  /**
   * Is generator function
   */
  generator: boolean;

  /**
   * Is async function
   */
  async: boolean;

  /**
   * Directive strings (e.g., "use strict", "use memo")
   */
  directives: Array<string>;
}
```

### HIR (Control Flow Graph)

Control flow graph representing function execution paths.

```typescript { .api }
/**
 * Control flow graph
 */
interface HIR {
  /**
   * Entry block ID (starting point of execution)
   */
  entry: BlockId;

  /**
   * Map of block IDs to basic blocks
   * Blocks are stored in reverse postorder for efficient traversal
   */
  blocks: Map<BlockId, BasicBlock>;
}
```

### BasicBlock

Basic block in the control flow graph.

```typescript { .api }
/**
 * Basic block - sequence of instructions with single entry and exit
 */
interface BasicBlock {
  /**
   * Block kind
   * - 'block': Regular block
   * - 'value': Block that produces a value
   * - 'loop': Loop block
   * - 'sequence': Sequence of statements
   * - 'catch': Exception handler block
   */
  kind: "block" | "value" | "loop" | "sequence" | "catch";

  /**
   * Unique block identifier
   */
  id: BlockId;

  /**
   * Sequence of instructions in the block
   */
  instructions: Array<Instruction>;

  /**
   * Terminal node (control flow transfer at block end)
   */
  terminal: Terminal;

  /**
   * Set of predecessor block IDs
   */
  preds: Set<BlockId>;

  /**
   * Phi nodes for SSA form
   */
  phis: Set<Phi>;
}
```

### Instruction

Instruction in HIR representing a single operation.

```typescript { .api }
/**
 * Single instruction with result and operation
 */
interface Instruction {
  /**
   * Unique instruction identifier
   */
  id: InstructionId;

  /**
   * Left-hand side (result storage location)
   */
  lvalue: Place;

  /**
   * Instruction value (operation and operands)
   */
  value: InstructionValue;

  /**
   * Source location
   */
  loc: SourceLocation;
}
```

### InstructionValue

Union type of all possible instruction operations (40+ types).

```typescript { .api }
/**
 * Instruction value types (union of 40+ instruction types)
 */
type InstructionValue =
  // Literals and variables
  | Primitive
  | LoadLocal
  | LoadGlobal
  | StoreGlobal
  // Property operations
  | PropertyLoad
  | PropertyStore
  | PropertyDelete
  // Function calls
  | CallExpression
  | MethodCall
  | NewExpression
  // Expressions
  | ArrayExpression
  | ObjectExpression
  | JsxExpression
  | BinaryExpression
  | UnaryExpression
  | UpdateExpression
  // Control flow
  | FunctionExpression
  | Destructure
  | StartMemoize
  | FinishMemoize
  // ... and many more
  ;
```

**Key Instruction Types:**

- **Primitive**: Literal values (numbers, strings, booleans, null, undefined)
- **LoadLocal**: Load local variable
- **LoadGlobal**: Load global variable
- **StoreGlobal**: Store to global variable
- **PropertyLoad**: Property access (`obj.prop` or `obj[key]`)
- **PropertyStore**: Property assignment (`obj.prop = value`)
- **PropertyDelete**: Property deletion (`delete obj.prop`)
- **CallExpression**: Function call
- **MethodCall**: Method call (`obj.method()`)
- **NewExpression**: Constructor call (`new Class()`)
- **ArrayExpression**: Array literal `[...]`
- **ObjectExpression**: Object literal `{...}`
- **JsxExpression**: JSX element `<Component />`
- **BinaryExpression**: Binary operations (`+`, `-`, `*`, etc.)
- **UnaryExpression**: Unary operations (`!`, `-`, `typeof`, etc.)
- **UpdateExpression**: Increment/decrement (`++`, `--`)
- **FunctionExpression**: Function expression or arrow function
- **Destructure**: Destructuring assignment
- **StartMemoize**: Begin memoization scope
- **FinishMemoize**: End memoization scope

### Terminal

Terminal node representing control flow transfer at block end (16+ types).

```typescript { .api }
/**
 * Terminal nodes for control flow
 */
type Terminal =
  | ReturnTerminal
  | ThrowTerminal
  | GotoTerminal
  | IfTerminal
  | SwitchTerminal
  | BranchTerminal
  | ForTerminal
  | ForOfTerminal
  | ForInTerminal
  | WhileTerminal
  | DoWhileTerminal
  | TryTerminal
  | LogicalTerminal
  | TernaryTerminal
  | OptionalTerminal
  | LabelTerminal
  | SequenceTerminal
  | ReactiveScopeTerminal
  | PrunedScopeTerminal
  | MaybeThrowTerminal
  | UnsupportedTerminal
  | UnreachableTerminal;
```

**Key Terminal Types:**

- **ReturnTerminal**: Function return statement
- **ThrowTerminal**: Throw exception
- **GotoTerminal**: Unconditional jump to block
- **IfTerminal**: Conditional branch (if/else)
- **SwitchTerminal**: Switch statement
- **BranchTerminal**: Generic branch
- **ForTerminal**: For loop
- **ForOfTerminal**: For-of loop
- **ForInTerminal**: For-in loop
- **WhileTerminal**: While loop
- **DoWhileTerminal**: Do-while loop
- **TryTerminal**: Try-catch-finally
- **LogicalTerminal**: Logical expression (`&&`, `||`)
- **TernaryTerminal**: Ternary conditional
- **OptionalTerminal**: Optional chaining
- **ReactiveScopeTerminal**: Reactive scope boundary
- **PrunedScopeTerminal**: Optimized-away scope
- **UnreachableTerminal**: Unreachable code

### Place

Storage location (variable, temporary, property).

```typescript { .api }
/**
 * Storage location in HIR
 */
interface Place {
  /**
   * Identifier for this place
   */
  identifier: Identifier;

  /**
   * Effect of accessing this place
   */
  effect: Effect;

  /**
   * Reactive dependencies
   */
  reactive: boolean;
}
```

### Identifier

Identifier with unique ID and metadata.

```typescript { .api }
/**
 * Identifier in HIR
 */
interface Identifier {
  /**
   * Unique identifier ID
   */
  id: IdentifierId;

  /**
   * Variable name (may be generated)
   */
  name: string;

  /**
   * Mutable location
   */
  mutableRange: MutableRange;

  /**
   * Scope where identifier is declared
   */
  scope: Scope | null;

  /**
   * Type information
   */
  type: Type;
}
```

### BlockId, InstructionId, IdentifierId

Unique identifier types.

```typescript { .api }
/**
 * Unique block identifier
 */
type BlockId = number & { readonly __brand: "BlockId" };

/**
 * Unique instruction identifier
 */
type InstructionId = number & { readonly __brand: "InstructionId" };

/**
 * Unique identifier ID
 */
type IdentifierId = number & { readonly __brand: "IdentifierId" };
```

### ID Creation Functions

Helper functions to create unique IDs.

```typescript { .api }
/**
 * Create block ID
 */
function makeBlockId(id: number): BlockId;

/**
 * Create instruction ID
 */
function makeInstructionId(id: number): InstructionId;

/**
 * Create identifier ID
 */
function makeIdentifierId(id: number): IdentifierId;

/**
 * Create identifier name from ID
 */
function makeIdentifierName(id: number): string;
```

## HIR Inspection Patterns

### Traverse All Blocks

```typescript
import { type HIR, type BasicBlock } from "babel-plugin-react-compiler";

function traverseHIR(hir: HIR) {
  const visited = new Set<number>();
  const queue = [hir.entry];

  while (queue.length > 0) {
    const blockId = queue.shift()!;

    if (visited.has(blockId)) {
      continue;
    }

    visited.add(blockId);
    const block = hir.blocks.get(blockId);

    if (!block) {
      continue;
    }

    console.log(`Block ${blockId} (${block.kind}):`);

    // Process instructions
    for (const instruction of block.instructions) {
      console.log(`  [${instruction.id}] ${instruction.lvalue.identifier.name} =`, instruction.value);
    }

    // Process terminal
    console.log(`  Terminal:`, block.terminal);

    // Add successor blocks to queue based on terminal
    // (implementation depends on terminal type)
  }
}
```

### Count Instructions by Type

```typescript
import { type HIRFunction } from "babel-plugin-react-compiler";

function countInstructionTypes(hirFn: HIRFunction): Map<string, number> {
  const counts = new Map<string, number>();

  for (const block of hirFn.body.blocks.values()) {
    for (const instruction of block.instructions) {
      const type = instruction.value.kind || instruction.value.type || "unknown";
      counts.set(type, (counts.get(type) || 0) + 1);
    }
  }

  return counts;
}
```

### Find Memoization Scopes

```typescript
import { type HIR } from "babel-plugin-react-compiler";

function findMemoizationScopes(hir: HIR): Array<{ start: number; end: number }> {
  const scopes: Array<{ start: number; end: number }> = [];
  let startInstruction: number | null = null;

  for (const block of hir.blocks.values()) {
    for (const instruction of block.instructions) {
      if (instruction.value.kind === "StartMemoize") {
        startInstruction = instruction.id;
      } else if (
        instruction.value.kind === "FinishMemoize" &&
        startInstruction !== null
      ) {
        scopes.push({ start: startInstruction, end: instruction.id });
        startInstruction = null;
      }
    }
  }

  return scopes;
}
```

## HIR Use Cases

### Debugging Compilation

Use `printHIR` to inspect compilation stages:

```typescript
import { run, printHIR } from "babel-plugin-react-compiler";

const pipeline = run(path, config, "Component", "c", null, filename, code);

for (const stage of pipeline) {
  if (stage.kind === "hir") {
    console.log(`\n=== HIR Stage: ${stage.name} ===`);
    console.log(printHIR(stage.value));
  }
}
```

### Custom Analysis

Build custom analysis passes over HIR:

```typescript
import { run, type HIRFunction } from "babel-plugin-react-compiler";

function analyzeComponentComplexity(hirFn: HIRFunction): {
  instructionCount: number;
  blockCount: number;
  branchCount: number;
} {
  let instructionCount = 0;
  let branchCount = 0;

  for (const block of hirFn.body.blocks.values()) {
    instructionCount += block.instructions.length;

    // Count branches
    const terminal = block.terminal;
    if (
      terminal.kind === "if" ||
      terminal.kind === "switch" ||
      terminal.kind === "ternary"
    ) {
      branchCount++;
    }
  }

  return {
    instructionCount,
    blockCount: hirFn.body.blocks.size,
    branchCount,
  };
}
```

### Extract Function Metadata

```typescript
import { type HIRFunction } from "babel-plugin-react-compiler";

function extractMetadata(hirFn: HIRFunction) {
  return {
    name: hirFn.id,
    type: hirFn.fnType,
    isAsync: hirFn.async,
    isGenerator: hirFn.generator,
    paramCount: hirFn.params.length,
    contextVars: hirFn.context.map((p) => p.identifier.name),
    directives: hirFn.directives,
    hasEffects: hirFn.effects !== null && hirFn.effects.length > 0,
  };
}
```
