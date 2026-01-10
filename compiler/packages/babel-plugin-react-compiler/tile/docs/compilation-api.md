# Compilation API

Programmatic API for compiling React code with the React Compiler. These functions provide direct access to the compilation pipeline for custom build tools and integrations.

## Capabilities

### Run Babel Plugin React Compiler

Run the compiler on source text without Babel configuration.

```typescript { .api }
/**
 * Run the compiler on source text
 * @param text - Source code to compile
 * @param file - Filename for error reporting and source maps
 * @param language - Source language ('flow' or 'typescript')
 * @param options - Plugin options (partial, merged with defaults)
 * @param includeAst - Include AST in result (default: false)
 * @returns Babel file result with compiled code, source maps, and optionally AST
 */
function runBabelPluginReactCompiler(
  text: string,
  file: string,
  language: "flow" | "typescript",
  options: Partial<PluginOptions> | null,
  includeAst?: boolean
): BabelCore.BabelFileResult;
```

**Usage Example:**

```typescript
import { runBabelPluginReactCompiler } from "babel-plugin-react-compiler";

const sourceCode = `
function Counter() {
  const [count, setCount] = useState(0);
  const doubled = count * 2;

  return (
    <div>
      <p>{doubled}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
`;

const result = runBabelPluginReactCompiler(
  sourceCode,
  "Counter.tsx",
  "typescript",
  {
    compilationMode: "infer",
    panicThreshold: "all_errors",
  },
  false // Don't include AST
);

console.log(result.code); // Optimized code
console.log(result.map); // Source map
```

### Compile Function

Compile a single function with full control over configuration.

```typescript { .api }
/**
 * Compile a single function
 * @param func - Babel NodePath to function node
 * @param config - Full environment configuration
 * @param fnType - Function type ('Component', 'Hook', or 'Other')
 * @param useMemoCacheIdentifier - Identifier for memo cache
 * @param logger - Logger instance for events (null to disable)
 * @param filename - Source filename for error reporting (null if unknown)
 * @param code - Original source code for error reporting (null if unavailable)
 * @returns Compiled function with AST and metadata
 */
function compile(
  func: NodePath<t.Function>,
  config: EnvironmentConfig,
  fnType: ReactFunctionType,
  useMemoCacheIdentifier: string,
  logger: Logger | null,
  filename: string | null,
  code: string | null
): CodegenFunction;
```

**Usage Example:**

```typescript
import { compile } from "babel-plugin-react-compiler";
import { parse } from "@babel/parser";
import traverse from "@babel/traverse";

const ast = parse(sourceCode, {
  sourceType: "module",
  plugins: ["typescript", "jsx"],
});

traverse(ast, {
  Function(path) {
    const compiled = compile(
      path,
      environmentConfig,
      "Component",
      "c",
      logger,
      "MyComponent.tsx",
      sourceCode
    );

    console.log("Memo slots used:", compiled.memoSlotsUsed);
    console.log("Memo blocks:", compiled.memoBlocks);
    console.log("Memoized values:", compiled.memoValues);
  },
});
```

### Compile Program

Compile an entire program (Babel plugin visitor usage).

```typescript { .api }
/**
 * Compile an entire program
 * @param program - Babel NodePath to program node
 * @param pass - Compiler pass state with configuration
 */
function compileProgram(
  program: NodePath<t.Program>,
  pass: CompilerPass
): void;
```

**Usage Example:**

```typescript
import { compileProgram } from "babel-plugin-react-compiler";
import { parse } from "@babel/parser";
import traverse from "@babel/traverse";

const ast = parse(sourceCode, {
  sourceType: "module",
  plugins: ["typescript", "jsx"],
});

traverse(ast, {
  Program(path) {
    const compilerPass: CompilerPass = {
      opts: pluginOptions,
      file: babelFile,
      filename: "app.tsx",
    };

    compileProgram(path, compilerPass);
  },
});
```

### Run Compilation Pipeline

Generator function that yields intermediate compilation stages.

```typescript { .api }
/**
 * Generator function for compilation pipeline stages
 * Yields pipeline values for each stage: AST, HIR, Reactive, Debug
 * @param func - Babel NodePath to function node
 * @param config - Full environment configuration
 * @param fnType - Function type ('Component', 'Hook', or 'Other')
 * @param useMemoCacheIdentifier - Identifier for memo cache
 * @param logger - Logger instance for events (null to disable)
 * @param filename - Source filename for error reporting (null if unknown)
 * @param code - Original source code for error reporting (null if unavailable)
 * @yields CompilerPipelineValue for each compilation stage
 * @returns Final compiled function
 */
function* run(
  func: NodePath<t.Function>,
  config: EnvironmentConfig,
  fnType: ReactFunctionType,
  useMemoCacheIdentifier: string,
  logger: Logger | null,
  filename: string | null,
  code: string | null
): Generator<CompilerPipelineValue, CodegenFunction>;
```

**Usage Example:**

```typescript
import { run } from "babel-plugin-react-compiler";
import { parse } from "@babel/parser";
import traverse from "@babel/traverse";

const ast = parse(sourceCode, {
  sourceType: "module",
  plugins: ["typescript", "jsx"],
});

traverse(ast, {
  Function(path) {
    const pipeline = run(
      path,
      environmentConfig,
      "Component",
      "c",
      null,
      "MyComponent.tsx",
      sourceCode
    );

    // Iterate through compilation stages
    for (const stage of pipeline) {
      if (stage.kind === "hir") {
        console.log("HIR stage:", stage.name);
        console.log(printHIR(stage.value));
      } else if (stage.kind === "reactive") {
        console.log("Reactive stage:", stage.name);
        console.log(printReactiveFunction(stage.value));
      } else if (stage.kind === "debug") {
        console.log("Debug info:", stage.value);
      }
    }

    // Get final result (last yielded value is CodegenFunction)
    const compiled = pipeline.return().value;
    console.log("Compilation complete");
  },
});
```

## Types

### CodegenFunction

The compiled output with metadata.

```typescript { .api }
interface CodegenFunction {
  /**
   * Type marker
   */
  type: "CodegenFunction";

  /**
   * Function identifier (null for anonymous)
   */
  id: t.Identifier | null;

  /**
   * Function parameters
   */
  params: t.FunctionDeclaration["params"];

  /**
   * Function body
   */
  body: t.BlockStatement;

  /**
   * Is generator function
   */
  generator: boolean;

  /**
   * Is async function
   */
  async: boolean;

  /**
   * Source location
   */
  loc: SourceLocation;

  /**
   * Total memo cache slots used
   * Number of $[0], $[1], etc. slots
   */
  memoSlotsUsed: number;

  /**
   * Number of reactive scopes (memo blocks)
   */
  memoBlocks: number;

  /**
   * Number of memoized values
   */
  memoValues: number;

  /**
   * Number of pruned scopes (hooks)
   * Scopes that were optimized away
   */
  prunedMemoBlocks: number;

  /**
   * Values in pruned scopes
   */
  prunedMemoValues: number;

  /**
   * Outlined functions
   * Functions extracted from the original
   */
  outlined: Array<{
    fn: CodegenFunction;
    type: ReactFunctionType | null;
  }>;
}
```

### CompilerPipelineValue

Union type representing compilation pipeline stages.

```typescript { .api }
type CompilerPipelineValue =
  | {
      kind: "ast";
      name: string;
      value: CodegenFunction;
    }
  | {
      kind: "hir";
      name: string;
      value: HIRFunction;
    }
  | {
      kind: "reactive";
      name: string;
      value: ReactiveFunction;
    }
  | {
      kind: "debug";
      name: string;
      value: string;
    };
```

### ReactFunctionType

Function type classification.

```typescript { .api }
type ReactFunctionType = "Component" | "Hook" | "Other";
```

### CompilerPass

Compiler pass state for Babel plugin integration.

```typescript { .api }
interface CompilerPass {
  /**
   * Plugin options
   */
  opts: PluginOptions;

  /**
   * Babel file state
   */
  file: BabelFile;

  /**
   * Source filename
   */
  filename: string;
}
```

## Compilation Metrics

After compilation, the `CodegenFunction` contains metrics about the optimization:

```typescript
const result = compile(path, config, "Component", "c", null, null, null);

console.log(`Optimization metrics:
  - Memo slots: ${result.memoSlotsUsed}
  - Memo blocks: ${result.memoBlocks}
  - Memoized values: ${result.memoValues}
  - Pruned blocks: ${result.prunedMemoBlocks}
  - Pruned values: ${result.prunedMemoValues}
  - Outlined functions: ${result.outlined.length}
`);
```

These metrics help understand:
- **memoSlotsUsed**: Memory overhead of memoization cache
- **memoBlocks**: Number of reactive scopes created
- **memoValues**: Total values being memoized
- **prunedMemoBlocks**: Optimized away scopes (e.g., for hooks)
- **prunedMemoValues**: Values in optimized away scopes
- **outlined**: Extracted helper functions

## Error Handling

All compilation functions may throw `CompilerError`:

```typescript
import { compile, CompilerError } from "babel-plugin-react-compiler";

try {
  const result = compile(path, config, "Component", "c", null, null, code);
  console.log("Compilation successful");
} catch (error) {
  if (error instanceof CompilerError) {
    console.error("Compilation errors:");
    for (const detail of error.details) {
      console.error(`  [${detail.severity}] ${detail.reason}`);
      if (detail.loc) {
        console.error(
          `    at ${detail.loc.filename}:${detail.loc.start.line}:${detail.loc.start.column}`
        );
      }
      if (detail.description) {
        console.error(`    ${detail.description}`);
      }
    }
  } else {
    throw error;
  }
}
```

## Integration Patterns

### Custom Build Tool Integration

```typescript
import {
  runBabelPluginReactCompiler,
  type PluginOptions,
} from "babel-plugin-react-compiler";
import { readFileSync } from "fs";

function compileFi(filename: string, options: Partial<PluginOptions>) {
  const source = readFileSync(filename, "utf-8");
  const language = filename.endsWith(".tsx") ? "typescript" : "flow";

  try {
    const result = runBabelPluginReactCompiler(
      source,
      filename,
      language,
      options
    );

    return {
      code: result.code,
      map: result.map,
      success: true,
    };
  } catch (error) {
    return {
      code: null,
      map: null,
      success: false,
      error,
    };
  }
}
```

### Custom Babel Plugin

```typescript
import { compileProgram, parsePluginOptions } from "babel-plugin-react-compiler";
import type { PluginObj } from "@babel/core";

export default function myCustomPlugin(): PluginObj {
  return {
    name: "my-custom-react-compiler",
    visitor: {
      Program(path, state) {
        const options = parsePluginOptions(state.opts);

        const compilerPass = {
          opts: options,
          file: state.file,
          filename: state.filename || "unknown",
        };

        compileProgram(path, compilerPass);
      },
    },
  };
}
```

### Pipeline Inspection

```typescript
import { run, printHIR, printReactiveFunction } from "babel-plugin-react-compiler";

function inspectCompilation(path, config) {
  const stages = [];
  const pipeline = run(path, config, "Component", "c", null, null, null);

  for (const stage of pipeline) {
    stages.push({
      kind: stage.kind,
      name: stage.name,
      output:
        stage.kind === "hir"
          ? printHIR(stage.value)
          : stage.kind === "reactive"
          ? printReactiveFunction(stage.value)
          : stage.kind === "debug"
          ? stage.value
          : null,
    });
  }

  return stages;
}
```
