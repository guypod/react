# Babel Plugin React Compiler

The Babel Plugin React Compiler is an optimizing compiler for React applications that automatically analyzes components and hooks to determine optimal memoization boundaries. It eliminates the need for manual `useMemo`, `useCallback`, and `React.memo` optimizations by generating optimized JavaScript that ensures only minimal parts re-render when state changes. The plugin validates that components and hooks follow the Rules of React during compilation and integrates seamlessly into existing Babel-based build pipelines.

## Package Information

- **Package Name**: babel-plugin-react-compiler
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: `npm install babel-plugin-react-compiler`
- **Documentation**: https://react.dev/learn/react-compiler

## Core Imports

ES Module imports:

```typescript
import BabelPluginReactCompiler from "babel-plugin-react-compiler";
import {
  runBabelPluginReactCompiler,
  compile,
  compileProgram,
  parsePluginOptions,
  run,
  CompilerError,
  ErrorSeverity,
  Effect,
  ValueKind,
  parseConfigPragma,
  printHIR,
  validateEnvironmentConfig,
  printReactiveFunction,
  type CompilerPipelineValue,
  type PluginOptions,
  type EnvironmentConfig,
  type ExternalFunction,
  type Hook,
  type SourceLocation,
  type CompilerErrorDetailOptions,
  type CompilerSuggestionOperation,
} from "babel-plugin-react-compiler";
```

CommonJS imports:

```javascript
const BabelPluginReactCompiler = require("babel-plugin-react-compiler");
const {
  runBabelPluginReactCompiler,
  compile,
  compileProgram,
  parsePluginOptions,
  // ... other exports
} = require("babel-plugin-react-compiler");
```

## Basic Usage

### As a Babel Plugin

Add to your Babel configuration:

```javascript
// babel.config.js
module.exports = {
  plugins: [
    ["babel-plugin-react-compiler", {
      compilationMode: "infer",
      panicThreshold: "all_errors",
    }]
  ]
};
```

### Programmatic Compilation

```typescript
import { runBabelPluginReactCompiler } from "babel-plugin-react-compiler";

const sourceCode = `
function MyComponent({ value }) {
  const doubled = value * 2;
  return <div>{doubled}</div>;
}
`;

const result = runBabelPluginReactCompiler(
  sourceCode,
  "MyComponent.tsx",
  "typescript",
  { compilationMode: "infer" }
);

console.log(result.code);
```

## Architecture

The React Compiler operates through a multi-stage pipeline:

1. **Parsing & Validation** - Parse source code and validate against React rules
2. **HIR Generation** - Convert to High-level Intermediate Representation with control flow graphs
3. **Reactive Scope Analysis** - Identify reactive scopes and dependencies
4. **Code Generation** - Generate optimized JavaScript with automatic memoization

Key architectural components:

- **Babel Plugin Interface** - Integrates with Babel's transformation pipeline
- **HIR (High-level IR)** - Control flow graph representation with instructions and terminals
- **Reactive Scopes** - Tracks dependencies and determines memoization boundaries
- **Type System** - Tracks effects and value kinds for optimization decisions
- **Error System** - Detailed error reporting with suggestions and source locations
- **Environment Config** - 70+ configuration options for customizing compiler behavior

## Capabilities

### Babel Plugin Integration

The default export provides a standard Babel plugin for adding to Babel configurations.

```typescript { .api }
/**
 * Default export - Babel plugin for React Compiler
 * @param babel - Babel core object
 * @returns Babel plugin object with visitor methods
 */
export default function BabelPluginReactCompiler(
  babel: typeof BabelCore
): BabelCore.PluginObj;
```

[Plugin Configuration](./plugin-configuration.md)

### Compilation Functions

Programmatic API for compiling React code directly without Babel configuration.

```typescript { .api }
/**
 * Run the compiler on source text
 * @param text - Source code to compile
 * @param file - Filename for error reporting
 * @param language - Source language ('flow' | 'typescript')
 * @param options - Plugin options (partial)
 * @param includeAst - Include AST in result (default: false)
 * @returns Babel file result with compiled code
 */
function runBabelPluginReactCompiler(
  text: string,
  file: string,
  language: "flow" | "typescript",
  options: Partial<PluginOptions> | null,
  includeAst?: boolean
): BabelCore.BabelFileResult;

/**
 * Compile a single function
 * @returns Compiled function with metadata
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

/**
 * Compile an entire program
 */
function compileProgram(
  program: NodePath<t.Program>,
  pass: CompilerPass
): void;

/**
 * Generator function for compilation pipeline stages
 * @yields Pipeline values for each compilation stage
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

[Compilation API](./compilation-api.md)

### Error Handling

Comprehensive error system with detailed diagnostics, source locations, and fix suggestions.

```typescript { .api }
/**
 * Main error class for compilation errors
 */
class CompilerError extends Error {
  details: Array<CompilerErrorDetail>;

  static invariant(
    condition: unknown,
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): asserts condition;

  static throwInvalidJS(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;

  static throwInvalidReact(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;

  hasErrors(): boolean;
  isCritical(): boolean;
}

/**
 * Error severity levels
 */
enum ErrorSeverity {
  InvalidJS = "InvalidJS",
  InvalidReact = "InvalidReact",
  InvalidConfig = "InvalidConfig",
  CannotPreserveMemoization = "CannotPreserveMemoization",
  Todo = "Todo",
  Invariant = "Invariant",
}
```

[Error Handling](./error-handling.md)

### HIR (High-level Intermediate Representation)

Control flow graph representation used internally for analysis and optimization.

```typescript { .api }
/**
 * Print HIR to string for debugging
 */
function printHIR(fn: HIRFunction): string;

/**
 * HIR function representation
 */
interface HIRFunction {
  loc: SourceLocation;
  id: string | null;
  fnType: ReactFunctionType; // 'Component' | 'Hook' | 'Other'
  env: Environment;
  params: Array<Place | SpreadPattern>;
  body: HIR; // Control flow graph
  generator: boolean;
  async: boolean;
}

/**
 * Control flow graph
 */
interface HIR {
  entry: BlockId;
  blocks: Map<BlockId, BasicBlock>;
}
```

[HIR Representation](./hir-representation.md)

### Reactive Scopes

Reactive function representation showing memoization boundaries and dependencies.

```typescript { .api }
/**
 * Print reactive function to string for debugging
 */
function printReactiveFunction(fn: ReactiveFunction): string;

/**
 * Reactive function with scope information
 */
interface ReactiveFunction {
  loc: SourceLocation;
  id: string | null;
  params: Array<Place | SpreadPattern>;
  body: ReactiveBlock;
  env: Environment;
}
```

[Reactive Scopes](./reactive-scopes.md)

### Type System

Effect tracking and value kind system for optimization decisions.

```typescript { .api }
/**
 * Effect types for tracking mutations
 */
enum Effect {
  Unknown = "<unknown>",
  Freeze = "freeze",
  Read = "read",
  Capture = "capture",
  ConditionallyMutate = "mutate?",
  Mutate = "mutate",
  Store = "store",
}

/**
 * Value kinds for optimization
 */
enum ValueKind {
  MaybeFrozen = "maybefrozen",
  Frozen = "frozen",
  Primitive = "primitive",
  Global = "global",
  Mutable = "mutable",
  Context = "context",
}
```

[Type System](./type-system.md)

### Configuration Parsing & Validation

Parse and validate configuration from various sources.

```typescript { .api }
/**
 * Parse plugin options from object
 * @throws CompilerError if invalid
 */
function parsePluginOptions(obj: unknown): PluginOptions;

/**
 * Parse configuration from pragma comments
 * @example "@react-forget {\"validateHooksUsage\": true}"
 */
function parseConfigPragma(pragma: string): EnvironmentConfig;

/**
 * Validate environment configuration
 * @returns Result with validated config or Zod error
 */
function validateEnvironmentConfig(
  config: unknown
): Result<EnvironmentConfig, ZodError>;
```

[Plugin Configuration](./plugin-configuration.md)

### Advanced API

Advanced functions for custom Babel plugin development and AST manipulation.

```typescript { .api }
/**
 * Insert gated function with feature flag
 */
function insertGatedFunctionDeclaration(
  fnPath: NodePath<t.FunctionDeclaration | t.ArrowFunctionExpression | t.FunctionExpression>,
  compiled: t.FunctionDeclaration | t.ArrowFunctionExpression | t.FunctionExpression,
  gating: ExternalFunction
): void;

/**
 * Add imports to program
 */
function addImportsToProgram(
  path: NodePath<t.Program>,
  importList: Array<ExternalFunction>
): void;

/**
 * Update memo cache function import
 */
function updateMemoCacheFunctionImport(
  program: NodePath<t.Program>,
  moduleName: string,
  useMemoCacheIdentifier: string
): void;

/**
 * Find ESLint/Flow suppressions in program
 */
function findProgramSuppressions(
  programComments: Array<t.Comment>,
  ruleNames: Array<string>,
  flowSuppressions: boolean
): Array<SuppressionRange>;

/**
 * Filter suppressions affecting a function
 */
function filterSuppressionsThatAffectFunction(
  suppressionRanges: Array<SuppressionRange>,
  fn: NodePath<t.Function>
): Array<SuppressionRange>;

/**
 * Convert suppressions to compiler error
 */
function suppressionsToCompilerError(
  suppressionRanges: Array<SuppressionRange>
): CompilerError | null;
```

[Advanced API](./advanced-api.md)

### Logging & Events

Logger interface for tracking compilation events, errors, and metrics.

```typescript { .api }
/**
 * Logger interface for compilation events
 */
interface Logger {
  logEvent(filename: string | null, event: LoggerEvent): void;
}

/**
 * Compilation events
 */
type LoggerEvent =
  | {
      kind: "CompileError";
      fnLoc: t.SourceLocation | null;
      detail: CompilerErrorDetailOptions;
    }
  | {
      kind: "CompileSuccess";
      fnLoc: t.SourceLocation | null;
      fnName: string | null;
      memoSlots: number;
      memoBlocks: number;
      memoValues: number;
    };
```

[Logging & Events](./logging.md)

## Types

### CompilerPipelineValue

Union type representing different stages in the compilation pipeline:

```typescript { .api }
type CompilerPipelineValue =
  | { kind: "ast"; name: string; value: CodegenFunction }
  | { kind: "hir"; name: string; value: HIRFunction }
  | { kind: "reactive"; name: string; value: ReactiveFunction }
  | { kind: "debug"; name: string; value: string };
```

### SourceLocation

Source code location for error reporting:

```typescript { .api }
interface SourceLocation {
  start: { line: number; column: number };
  end: { line: number; column: number };
  filename?: string | null;
}

/**
 * GeneratedSource symbol
 * Special symbol indicating compiler-generated source (not from original code)
 * Used in SourceLocation when location is not from user code
 */
const GeneratedSource: unique symbol;
```

### ExternalFunction

Reference to an external function for feature gating or instrumentation:

```typescript { .api }
interface ExternalFunction {
  source: string; // Import source module
  importSpecifierName: string; // Function name to import
}
```

### Hook

Configuration for custom hooks:

```typescript { .api }
interface Hook {
  effectKind: Effect; // Effect on arguments
  valueKind: ValueKind; // Return value kind
  noAlias: boolean; // Whether aliasing is prevented
  transitiveMixedData: boolean; // Returns JSON-like data
}
```
