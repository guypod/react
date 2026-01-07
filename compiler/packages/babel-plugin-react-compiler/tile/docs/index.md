# babel-plugin-react-compiler

The React Compiler is a Babel plugin that automatically optimizes React applications by ensuring only minimal parts of components and hooks re-render when state changes. The compiler analyzes React code at build time, validates that components and hooks follow the Rules of React, and generates optimized code that maintains correctness while improving performance through automatic memoization.

## Package Information

- **Package Name**: babel-plugin-react-compiler
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: `npm install babel-plugin-react-compiler`
- **Babel Version**: 7.x

## Core Imports

ESM:

```typescript
import BabelPluginReactCompiler from "babel-plugin-react-compiler";
import {
  runBabelPluginReactCompiler,
  compile,
  compileProgram,
  parsePluginOptions,
  CompilerError,
  ErrorSeverity,
  Effect,
  ValueKind,
} from "babel-plugin-react-compiler";
```

CommonJS:

```javascript
const BabelPluginReactCompiler = require("babel-plugin-react-compiler");
const {
  runBabelPluginReactCompiler,
  compile,
  compileProgram,
  CompilerError,
} = require("babel-plugin-react-compiler");
```

## Basic Usage

### As a Babel Plugin

```javascript
// babel.config.js
module.exports = {
  plugins: [
    [
      "babel-plugin-react-compiler",
      {
        compilationMode: "infer",
        panicThreshold: "none",
        environment: {
          enablePreserveExistingMemoizationGuarantees: false,
        },
      },
    ],
  ],
};
```

### Standalone Compilation

```javascript
import { runBabelPluginReactCompiler } from "babel-plugin-react-compiler";

const sourceCode = `
function MyComponent({ items }) {
  return <ul>{items.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
}
`;

const result = runBabelPluginReactCompiler(
  sourceCode,
  "MyComponent.tsx",
  "typescript",
  { compilationMode: "infer" }
);

console.log(result.code); // Optimized code with automatic memoization
```

## Architecture

The React Compiler operates through multiple compilation stages:

- **Parsing**: Source code is parsed into a Babel AST
- **HIR (High-level Intermediate Representation)**: AST is converted to HIR, a control flow graph representation
- **Reactive Scopes**: HIR is analyzed to identify reactive dependencies and create memoization scopes
- **Codegen**: Reactive scopes are converted back to optimized JavaScript/TypeScript with memoization

The compiler integrates seamlessly into Babel-based build pipelines and provides comprehensive error reporting, validation, and configuration options.

## Capabilities

### Babel Plugin

The default export is the main Babel plugin for integrating React Compiler into build pipelines.

```typescript { .api }
declare function BabelPluginReactCompiler(
  babel: typeof BabelCore
): BabelCore.PluginObj;

export default BabelPluginReactCompiler;
```

[Babel Plugin Usage](./babel-plugin.md)

### Standalone Compilation

Compile React code without a full Babel configuration setup, useful for testing and standalone tools.

```typescript { .api }
function runBabelPluginReactCompiler(
  text: string,
  file: string,
  language: "flow" | "typescript",
  options: Partial<PluginOptions> | null,
  includeAst?: boolean
): BabelCore.BabelFileResult;
```

[Compilation Functions](./compilation.md)

### Configuration

Comprehensive configuration options for controlling compilation behavior, validation rules, and optimization strategies.

```typescript { .api }
type PluginOptions = {
  environment: PartialEnvironmentConfig | null;
  logger: Logger | null;
  gating: ExternalFunction | null;
  panicThreshold: "all_errors" | "critical_errors" | "none";
  noEmit: boolean;
  compilationMode: "infer" | "syntax" | "annotation" | "all";
  runtimeModule?: string | null | undefined;
  eslintSuppressionRules?: Array<string> | null | undefined;
  flowSuppressions: boolean;
  ignoreUseNoForget: boolean;
  sources?: Array<string> | ((filename: string) => boolean) | null;
  enableReanimatedCheck: boolean;
};

function parsePluginOptions(obj: unknown): PluginOptions;
```

[Configuration Options](./configuration.md)

### Error Handling

Comprehensive error handling with detailed diagnostics, severity levels, and suggestions for fixing issues.

```typescript { .api }
class CompilerError extends Error {
  details: Array<CompilerErrorDetail>;
  name: string;
  message: string;

  push(options: CompilerErrorDetailOptions): CompilerErrorDetail;
  pushErrorDetail(detail: CompilerErrorDetail): CompilerErrorDetail;
  hasErrors(): boolean;
  isCritical(): boolean;
  toString(): string;

  static invariant(
    condition: unknown,
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): asserts condition;
  static throwTodo(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;
  static throwInvalidJS(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;
  static throwInvalidReact(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;
  static throwInvalidConfig(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;
  static throw(options: CompilerErrorDetailOptions): never;
}

enum ErrorSeverity {
  InvalidJS = "InvalidJS",
  InvalidReact = "InvalidReact",
  InvalidConfig = "InvalidConfig",
  CannotPreserveMemoization = "CannotPreserveMemoization",
  Todo = "Todo",
  Invariant = "Invariant",
}
```

[Error Handling](./errors.md)

### Type System

Type definitions for representing effects, value kinds, and compilation artifacts.

```typescript { .api }
enum Effect {
  Unknown = "<unknown>",
  Freeze = "freeze",
  Read = "read",
  Capture = "capture",
  ConditionallyMutate = "mutate?",
  Mutate = "mutate",
  Store = "store",
}

enum ValueKind {
  MaybeFrozen = "maybefrozen",
  Frozen = "frozen",
  Primitive = "primitive",
  Global = "global",
  Mutable = "mutable",
  Context = "context",
}

type CodegenFunction = {
  type: "CodegenFunction";
  id: t.Identifier | null;
  params: Array<
    t.Identifier | t.Pattern | t.RestElement | t.TSParameterProperty
  >;
  body: t.BlockStatement;
  generator: boolean;
  async: boolean;
  loc: SourceLocation;
  memoSlotsUsed: number;
  memoBlocks: number;
  memoValues: number;
  prunedMemoBlocks: number;
  prunedMemoValues: number;
  outlined: Array<{
    fn: CodegenFunction;
    type: ReactFunctionType | null;
  }>;
};
```

[Type System](./types.md)

### Utilities

Helper functions for debugging, printing intermediate representations, and configuration parsing.

```typescript { .api }
function printHIR(ir: HIR, options?: { indent?: number } | null): string;

function printReactiveFunction(fn: ReactiveFunction): string;

function parseConfigPragma(pragma: string): EnvironmentConfig;

function validateEnvironmentConfig(
  partialConfig: PartialEnvironmentConfig
): EnvironmentConfig;
```

[Utility Functions](./utilities.md)
