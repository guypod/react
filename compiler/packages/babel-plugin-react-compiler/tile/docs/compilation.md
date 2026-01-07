# Compilation Functions

Functions for compiling React code through the React Compiler pipeline, either standalone or as part of a larger program.

## Capabilities

### runBabelPluginReactCompiler

Standalone function to compile React code without a full Babel configuration setup. Useful for testing, standalone tools, and programmatic compilation.

```typescript { .api }
/**
 * Compiles React code using the React Compiler
 * @param text - Source code to compile
 * @param file - Filename/path for error reporting and source maps
 * @param language - Language variant to parse ('flow' or 'typescript')
 * @param options - Compiler configuration options
 * @param includeAst - Whether to include the AST in the result (default: false)
 * @returns Transformed code with optional AST
 */
function runBabelPluginReactCompiler(
  text: string,
  file: string,
  language: "flow" | "typescript",
  options: Partial<PluginOptions> | null,
  includeAst?: boolean
): BabelCore.BabelFileResult;
```

**Usage Examples:**

```typescript
import { runBabelPluginReactCompiler } from "babel-plugin-react-compiler";

// Basic TypeScript compilation
const result = runBabelPluginReactCompiler(
  `
function MyComponent({ count }) {
  return <div>Count: {count}</div>;
}
`,
  "MyComponent.tsx",
  "typescript",
  { compilationMode: "infer" }
);

console.log(result.code); // Optimized code

// Flow compilation
const flowResult = runBabelPluginReactCompiler(
  `
// @flow
function MyComponent({ count }: { count: number }) {
  return <div>Count: {count}</div>;
}
`,
  "MyComponent.js",
  "flow",
  { compilationMode: "all" }
);

// Include AST in result
const withAst = runBabelPluginReactCompiler(
  sourceCode,
  "MyComponent.tsx",
  "typescript",
  { compilationMode: "infer" },
  true // includeAst
);

console.log(withAst.ast); // Babel AST
```

**With Custom Configuration:**

```typescript
const result = runBabelPluginReactCompiler(
  sourceCode,
  "MyComponent.tsx",
  "typescript",
  {
    compilationMode: "all",
    noEmit: false,
    environment: {
      enablePreserveExistingMemoizationGuarantees: true,
      validateHooksUsage: true,
      enableMemoizationComments: true,
    },
    logger: {
      logEvent(filename, event) {
        if (event.kind === "CompileSuccess") {
          console.log(`✓ ${event.fnName}: ${event.memoBlocks} memo blocks`);
        }
      },
    },
  }
);
```

---

### compile

Compiles a single function through the entire React Compiler pipeline. This is a lower-level API that operates on Babel AST nodes.

```typescript { .api }
/**
 * Compiles a single function through the React Compiler pipeline
 * @param func - Babel AST node path for the function to compile
 * @param config - Environment configuration
 * @param fnType - Type of React function (Component/Hook/Other)
 * @param useMemoCacheIdentifier - Name for the memo cache variable
 * @param logger - Optional logger for compilation events
 * @param filename - Source filename for error reporting
 * @param code - Source code for HMR support
 * @returns Compiled function with memo blocks and statistics
 */
function compile(
  func: NodePath<
    t.FunctionDeclaration | t.ArrowFunctionExpression | t.FunctionExpression
  >,
  config: EnvironmentConfig,
  fnType: ReactFunctionType,
  useMemoCacheIdentifier: string,
  logger: Logger | null,
  filename: string | null,
  code: string | null
): CodegenFunction;

type ReactFunctionType = "Component" | "Hook" | "Other";
```

**Usage Example:**

```typescript
import { compile, validateEnvironmentConfig } from "babel-plugin-react-compiler";
import * as babel from "@babel/core";
import traverse from "@babel/traverse";

const code = `function MyComponent({ count }) { return <div>{count}</div>; }`;
const ast = babel.parse(code, { sourceType: "module", plugins: ["jsx"] });

traverse(ast, {
  FunctionDeclaration(path) {
    const config = validateEnvironmentConfig({
      enablePreserveExistingMemoizationGuarantees: false,
    });

    const compiled = compile(
      path,
      config,
      "Component",
      "useMemoCache",
      null,
      "MyComponent.js",
      code
    );

    console.log("Memo slots used:", compiled.memoSlotsUsed);
    console.log("Memo blocks:", compiled.memoBlocks);
    console.log("Memoized values:", compiled.memoValues);
  },
});
```

---

### compileProgram

Compiles all eligible functions in a Babel program/file. This is the function used internally by the Babel plugin.

```typescript { .api }
/**
 * Compiles all eligible functions in a program
 * @param program - The Babel Program node path
 * @param pass - Compiler pass configuration object
 */
function compileProgram(
  program: NodePath<t.Program>,
  pass: CompilerPass
): void;

type CompilerPass = {
  opts: PluginOptions;
  filename: string | null;
  comments: Array<t.CommentBlock | t.CommentLine>;
  code: string | null;
};
```

**Usage Example:**

```typescript
import { compileProgram, parsePluginOptions } from "babel-plugin-react-compiler";
import * as babel from "@babel/core";
import traverse from "@babel/traverse";

const code = `
function Component1({ a }) { return <div>{a}</div>; }
function Component2({ b }) { return <span>{b}</span>; }
`;

const ast = babel.parse(code, { sourceType: "module", plugins: ["jsx"] });

traverse(ast, {
  Program(path) {
    const pass = {
      opts: parsePluginOptions({ compilationMode: "infer" }),
      filename: "components.js",
      comments: [],
      code: code,
    };

    compileProgram(path, pass);
    // Both Component1 and Component2 are now compiled
  },
});
```

---

### run (Generator Function)

Generator function that yields intermediate compilation results for each pipeline stage. Useful for debugging, visualization, and understanding the compilation process.

```typescript { .api }
/**
 * Generator that yields intermediate compilation results
 * @param func - Babel AST node path for the function to compile
 * @param config - Environment configuration
 * @param fnType - Type of React function
 * @param useMemoCacheIdentifier - Name for memo cache variable
 * @param logger - Optional logger
 * @param filename - Source filename
 * @param code - Source code for HMR
 * @yields CompilerPipelineValue - Intermediate results at each stage
 * @returns CodegenFunction - Final compiled function
 */
function* run(
  func: NodePath<
    t.FunctionDeclaration | t.ArrowFunctionExpression | t.FunctionExpression
  >,
  config: EnvironmentConfig,
  fnType: ReactFunctionType,
  useMemoCacheIdentifier: string,
  logger: Logger | null,
  filename: string | null,
  code: string | null
): Generator<CompilerPipelineValue, CodegenFunction>;

type CompilerPipelineValue =
  | { kind: "ast"; name: string; value: CodegenFunction }
  | { kind: "hir"; name: string; value: HIRFunction }
  | { kind: "reactive"; name: string; value: ReactiveFunction }
  | { kind: "debug"; name: string; value: string };
```

**Usage Example:**

```typescript
import { run, validateEnvironmentConfig, printHIR, printReactiveFunction } from "babel-plugin-react-compiler";
import * as babel from "@babel/core";
import traverse from "@babel/traverse";

const code = `function MyComponent({ items }) {
  return <ul>{items.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
}`;

const ast = babel.parse(code, { sourceType: "module", plugins: ["jsx", "typescript"] });

traverse(ast, {
  FunctionDeclaration(path) {
    const config = validateEnvironmentConfig({});
    const generator = run(
      path,
      config,
      "Component",
      "useMemoCache",
      null,
      "MyComponent.tsx",
      code
    );

    // Iterate through compilation stages
    for (const stage of generator) {
      console.log(`\n=== Stage: ${stage.name} (${stage.kind}) ===`);

      if (stage.kind === "hir") {
        console.log(printHIR(stage.value));
      } else if (stage.kind === "reactive") {
        console.log(printReactiveFunction(stage.value));
      } else if (stage.kind === "debug") {
        console.log(stage.value);
      } else if (stage.kind === "ast") {
        console.log("Final codegen:", stage.value);
      }
    }
  },
});
```

**Pipeline Stages:**

The generator yields values at these stages:

1. **HIR stages** - High-level intermediate representation
   - BuildHIR
   - Enter SSA
   - Various optimization passes
2. **Reactive stages** - Reactive scope analysis
   - BuildReactiveBlocks
   - PruneNonReactiveDependencies
   - MergeOverlappingReactiveScopes
3. **Codegen stages** - Final code generation
   - CodegenReactiveFunction
