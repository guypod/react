# Type System

Type definitions for representing effects, value kinds, compilation artifacts, and intermediate representations.

## Capabilities

### Effect

Enum defining effects that operations can have on values during compilation analysis.

```typescript { .api }
enum Effect {
  /**
   * Unknown effect (default)
   * Not allowed after type inference completes
   */
  Unknown = "<unknown>",

  /**
   * Freezes the value
   * Value becomes immutable after this operation
   */
  Freeze = "freeze",

  /**
   * Reads the value
   * Value is accessed but not modified
   */
  Read = "read",

  /**
   * Reads and stores the value
   * Value is captured for later use
   */
  Capture = "capture",

  /**
   * May mutate the value
   * Value might be modified depending on control flow
   */
  ConditionallyMutate = "mutate?",

  /**
   * Does mutate the value
   * Value is definitely modified
   */
  Mutate = "mutate",

  /**
   * May alias the value
   * Value might be stored with potential aliasing
   */
  Store = "store",
}
```

**Usage Example:**

```typescript
import { Effect } from "babel-plugin-react-compiler";

// Define custom hook with specific effect
const customHook = {
  effectKind: Effect.Read, // Hook reads arguments
  valueKind: "Mutable",
  noAlias: false,
  transitiveMixedData: false,
};

// Categorize effects
function getEffectCategory(effect: Effect): string {
  switch (effect) {
    case Effect.Unknown:
      return "Unknown";
    case Effect.Freeze:
    case Effect.Read:
      return "Safe - No mutation";
    case Effect.Capture:
    case Effect.Store:
      return "Aliasing";
    case Effect.ConditionallyMutate:
    case Effect.Mutate:
      return "Mutating";
  }
}
```

---

### ValueKind

Enum defining categories of value mutability and behavior.

```typescript { .api }
enum ValueKind {
  /**
   * May or may not be frozen
   * Mutability is uncertain
   */
  MaybeFrozen = "maybefrozen",

  /**
   * Definitely frozen (immutable)
   * Value cannot be mutated
   */
  Frozen = "frozen",

  /**
   * Primitive value
   * number, string, boolean, null, undefined, symbol, bigint
   */
  Primitive = "primitive",

  /**
   * Global value
   * From global scope (window, document, etc.)
   */
  Global = "global",

  /**
   * Mutable value
   * Can be modified
   */
  Mutable = "mutable",

  /**
   * Context value
   * From React Context
   */
  Context = "context",
}
```

**Usage Example:**

```typescript
import { ValueKind, Effect } from "babel-plugin-react-compiler";

// Define hooks with different value kinds
const hooks = {
  // Returns immutable data
  useImmutableData: {
    effectKind: Effect.Read,
    valueKind: ValueKind.Frozen,
    noAlias: false,
    transitiveMixedData: true,
  },

  // Returns mutable state
  useState: {
    effectKind: Effect.Store,
    valueKind: ValueKind.Mutable,
    noAlias: false,
    transitiveMixedData: false,
  },

  // Returns context value
  useContext: {
    effectKind: Effect.Read,
    valueKind: ValueKind.Context,
    noAlias: false,
    transitiveMixedData: false,
  },

  // Returns primitive
  useId: {
    effectKind: Effect.Read,
    valueKind: ValueKind.Primitive,
    noAlias: true,
    transitiveMixedData: false,
  },
};

// Check if value can be mutated
function isMutable(kind: ValueKind): boolean {
  return kind === ValueKind.Mutable || kind === ValueKind.MaybeFrozen;
}

// Check if value is guaranteed immutable
function isImmutable(kind: ValueKind): boolean {
  return (
    kind === ValueKind.Frozen ||
    kind === ValueKind.Primitive ||
    kind === ValueKind.Global
  );
}
```

---

### CodegenFunction

The final compiled function output with Babel AST and compilation statistics.

```typescript { .api }
type CodegenFunction = {
  /** Type tag */
  type: "CodegenFunction";

  /** Function identifier (name) */
  id: t.Identifier | null;

  /** Function parameters */
  params: Array<
    t.Identifier | t.Pattern | t.RestElement | t.TSParameterProperty
  >;

  /** Function body */
  body: t.BlockStatement;

  /** Whether function is a generator */
  generator: boolean;

  /** Whether function is async */
  async: boolean;

  /** Source location */
  loc: SourceLocation;

  /** Number of memo cache slots used */
  memoSlotsUsed: number;

  /** Number of reactive scopes (memo blocks) */
  memoBlocks: number;

  /** Number of memoized values */
  memoValues: number;

  /** Number of scopes discarded due to hooks */
  prunedMemoBlocks: number;

  /** Number of values that couldn't be memoized */
  prunedMemoValues: number;

  /** Outlined functions extracted from the main function */
  outlined: Array<{
    fn: CodegenFunction;
    type: ReactFunctionType | null;
  }>;
};

type ReactFunctionType = "Component" | "Hook" | "Other";
```

**Usage Example:**

```typescript
import { compile, validateEnvironmentConfig } from "babel-plugin-react-compiler";
import * as babel from "@babel/core";
import traverse from "@babel/traverse";

const code = `
function MyComponent({ items }) {
  const processed = items.map(item => item.value * 2);
  return <ul>{processed.map(v => <li key={v}>{v}</li>)}</ul>;
}
`;

const ast = babel.parse(code, { sourceType: "module", plugins: ["jsx"] });

traverse(ast, {
  FunctionDeclaration(path) {
    const config = validateEnvironmentConfig({});
    const compiled = compile(
      path,
      config,
      "Component",
      "useMemoCache",
      null,
      "MyComponent.js",
      code
    );

    // Access compilation statistics
    console.log("Compilation Statistics:");
    console.log(`  Memo slots: ${compiled.memoSlotsUsed}`);
    console.log(`  Memo blocks: ${compiled.memoBlocks}`);
    console.log(`  Memoized values: ${compiled.memoValues}`);
    console.log(`  Pruned blocks: ${compiled.prunedMemoBlocks}`);
    console.log(`  Pruned values: ${compiled.prunedMemoValues}`);

    // Check for outlined functions
    if (compiled.outlined.length > 0) {
      console.log(`\nOutlined functions: ${compiled.outlined.length}`);
      for (const { fn, type } of compiled.outlined) {
        console.log(`  - ${fn.id?.name || "anonymous"} (${type})`);
      }
    }

    // Generate code from AST
    const generator = require("@babel/generator").default;
    const output = generator(compiled.body);
    console.log("\nGenerated code:");
    console.log(output.code);
  },
});
```

---

### CompilerPipelineValue

Discriminated union representing intermediate values yielded during compilation pipeline stages.

```typescript { .api }
type CompilerPipelineValue =
  | {
      /** AST stage output */
      kind: "ast";
      /** Stage name */
      name: string;
      /** Compiled function */
      value: CodegenFunction;
    }
  | {
      /** HIR (High-level Intermediate Representation) stage */
      kind: "hir";
      /** Stage name */
      name: string;
      /** HIR function */
      value: HIRFunction;
    }
  | {
      /** Reactive scope analysis stage */
      kind: "reactive";
      /** Stage name */
      name: string;
      /** Reactive function */
      value: ReactiveFunction;
    }
  | {
      /** Debug output stage */
      kind: "debug";
      /** Stage name */
      name: string;
      /** Debug string */
      value: string;
    };
```

**Usage Example:**

```typescript
import {
  run,
  validateEnvironmentConfig,
  printHIR,
  printReactiveFunction,
} from "babel-plugin-react-compiler";
import * as babel from "@babel/core";
import traverse from "@babel/traverse";

const code = `
function MyComponent({ data }) {
  const filtered = data.filter(item => item.active);
  return <div>{filtered.length} active items</div>;
}
`;

const ast = babel.parse(code, { sourceType: "module", plugins: ["jsx"] });

traverse(ast, {
  FunctionDeclaration(path) {
    const config = validateEnvironmentConfig({});
    const generator = run(
      path,
      config,
      "Component",
      "useMemoCache",
      null,
      "MyComponent.js",
      code
    );

    // Iterate through all compilation stages
    for (const stage of generator) {
      console.log(`\n${"=".repeat(60)}`);
      console.log(`Stage: ${stage.name}`);
      console.log(`Kind: ${stage.kind}`);
      console.log("=".repeat(60));

      switch (stage.kind) {
        case "hir":
          // High-level intermediate representation
          console.log(printHIR(stage.value));
          break;

        case "reactive":
          // Reactive scope analysis
          console.log(printReactiveFunction(stage.value));
          break;

        case "debug":
          // Debug messages
          console.log(stage.value);
          break;

        case "ast":
          // Final AST output
          console.log("Final compilation complete");
          console.log(`Memo blocks: ${stage.value.memoBlocks}`);
          console.log(`Memo values: ${stage.value.memoValues}`);
          break;
      }
    }
  },
});
```

---

### SourceLocation

Type representing source location in original code or generated code.

```typescript { .api }
type SourceLocation = t.SourceLocation | typeof GeneratedSource;

/**
 * Symbol indicating generated code (not from original source)
 */
declare const GeneratedSource: unique symbol;
```

**t.SourceLocation structure:**

```typescript
type SourceLocation = {
  start: {
    line: number; // 1-indexed line number
    column: number; // 0-indexed column number
  };
  end: {
    line: number;
    column: number;
  };
  filename?: string;
  identifierName?: string;
};
```

**Usage Example:**

```typescript
import { CompilerError, ErrorSeverity } from "babel-plugin-react-compiler";

function createError(loc: SourceLocation) {
  const error = new CompilerError();

  // Check if location is from generated code
  if (loc === GeneratedSource) {
    error.push({
      reason: "Error in generated code",
      description: "This error occurred in compiler-generated code",
      severity: ErrorSeverity.Invariant,
      loc: null,
    });
  } else {
    // Real source location
    error.push({
      reason: "Error in source code",
      description: `Error at line ${loc.start.line}, column ${loc.start.column}`,
      severity: ErrorSeverity.InvalidJS,
      loc: loc,
    });
  }

  return error;
}

// Format location for display
function formatLocation(loc: SourceLocation | null): string {
  if (loc === null) {
    return "(unknown location)";
  }
  if (loc === GeneratedSource) {
    return "(generated code)";
  }
  return `${loc.filename || "unknown"}:${loc.start.line}:${loc.start.column}`;
}
```

---

### CompilerPass

Configuration object for program-level compilation.

```typescript { .api }
type CompilerPass = {
  /** Plugin options */
  opts: PluginOptions;

  /** Source filename */
  filename: string | null;

  /** Comments from source file */
  comments: Array<t.CommentBlock | t.CommentLine>;

  /** Source code */
  code: string | null;
};
```

**Usage Example:**

```typescript
import {
  compileProgram,
  parsePluginOptions,
} from "babel-plugin-react-compiler";
import * as babel from "@babel/core";
import traverse from "@babel/traverse";

const code = `
function Component1({ a }) { return <div>{a}</div>; }
function Component2({ b }) { return <span>{b}</span>; }
`;

const result = babel.parseSync(code, {
  sourceType: "module",
  plugins: ["jsx"],
});

if (result) {
  traverse(result, {
    Program(path) {
      const pass: CompilerPass = {
        opts: parsePluginOptions({
          compilationMode: "infer",
          environment: {
            validateHooksUsage: true,
          },
        }),
        filename: "components.js",
        comments: result.comments || [],
        code: code,
      };

      compileProgram(path, pass);
      // All eligible functions are now compiled
    },
  });
}
```

---

### HIRFunction

High-level Intermediate Representation of a function (internal type, yielded by pipeline).

```typescript { .api }
type HIRFunction = {
  /** Function identifier */
  id: Identifier;

  /** Function body as control flow graph */
  body: BasicBlock;

  /** Environment/scope information */
  env: Environment;

  /** Function parameters */
  params: Array<Place>;

  /** Return type */
  returnType: Type;

  /** Source location */
  loc: SourceLocation;

  /** Additional metadata */
  [key: string]: any;
};
```

---

### ReactiveFunction

Function with reactive scope analysis (internal type, yielded by pipeline).

```typescript { .api }
type ReactiveFunction = {
  /** Function identifier */
  id: Identifier;

  /** Reactive blocks */
  blocks: Array<ReactiveBlock>;

  /** Reactive scopes */
  scopes: Array<ReactiveScope>;

  /** Dependencies */
  dependencies: Array<Dependency>;

  /** Source location */
  loc: SourceLocation;

  /** Additional metadata */
  [key: string]: any;
};
```

**Usage with printReactiveFunction:**

```typescript
import { run, printReactiveFunction } from "babel-plugin-react-compiler";

const generator = run(/* ... */);

for (const stage of generator) {
  if (stage.kind === "reactive") {
    // Print reactive function with scopes
    const formatted = printReactiveFunction(stage.value);
    console.log(formatted);

    // Access reactive function properties
    console.log(`Scopes: ${stage.value.scopes.length}`);
    console.log(`Dependencies: ${stage.value.dependencies.length}`);
  }
}
```
