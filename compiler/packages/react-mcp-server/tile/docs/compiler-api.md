# Compiler API

React Compiler integration for automatic memoization and optimization of React components and hooks.

## Imports

```typescript
import { compile, lastResult, type PrintedCompilerPipelineValue } from 'react-mcp-server/src/compiler';
import type { BabelFileResult } from '@babel/core';
import type { PluginOptions } from 'babel-plugin-react-compiler/src';
```

## Capabilities

### Compile Function

Compiles React code using the React Compiler with Babel transformation and Prettier formatting.

```typescript { .api }
/**
 * Compile React code with React Compiler
 * @param options - Compilation options (destructured)
 * @returns Promise resolving to Babel file result with compiled code
 */
function compile({
  text,
  file,
  options,
}: CompileOptions): Promise<BabelFileResult>;

interface CompileOptions {
  /** Source code to compile */
  text: string;
  /** Filename for source maps and error reporting (e.g., 'anonymous.tsx') */
  file: string;
  /** React Compiler plugin options, or null for defaults */
  options: PluginOptions | null;
}

// Note: BabelFileResult is from @babel/core
interface BabelFileResult {
  /** Compiled code */
  code: string | null;
  /** Source map */
  map: object | null;
  /** AST (Abstract Syntax Tree) */
  ast: object | null;
  /** Babel metadata */
  metadata?: object;
}
```

**Compilation Pipeline**:

1. **Parse**: Parse source code with Babel using TypeScript and JSX plugins
   - Parser options: `['typescript', 'jsx']`
   - Source type: `'module'`

2. **Transform**: Transform AST with React Compiler plugin
   - Applies automatic memoization
   - Optimizes component re-renders
   - Generates compiler runtime imports

3. **Format**: Format output with Prettier
   - Parser: `'babel-ts'`
   - Options: `{ semi: false }`

4. **Store**: Save result to `lastResult` global variable

5. **Return**: Babel file result with code, source map, and metadata

**Usage Example**:

```typescript
import { compile } from 'react-mcp-server/src/compiler';

const result = await compile({
  text: `
    function MyComponent({ name }) {
      return <div>Hello {name}</div>;
    }
  `,
  file: 'MyComponent.tsx',
  options: null
});

console.log(result.code);
// Output includes:
// import { c as _c } from "react/compiler-runtime"
// const $ = _c(1)
// ... optimized code
```

**Error Handling**:
- Throws Error if parsing fails: `"Could not parse"`
- Throws Error if transformation fails with invalid result
- Logs to console.error if Prettier formatting fails (non-fatal)

---

### Compilation Result

Global variable storing the last compilation result for inspection or debugging.

```typescript { .api }
/**
 * Last compilation result from compile() function
 * Initially null, updated after each successful compilation
 */
let lastResult: BabelFileResult | null;
```

**Usage**: Access the most recent compilation result without calling compile again.

```typescript
import { compile, lastResult } from 'react-mcp-server/src/compiler';

await compile({ text: code, file: 'test.tsx', options: null });

// Access last result
if (lastResult?.code) {
  console.log('Last compiled code:', lastResult.code);
}
```

---

### Compiler Pipeline Value

Type representing intermediate compiler pass outputs for debugging and analysis.

```typescript { .api }
/**
 * Represents output from different compiler pipeline passes
 * Used for debugging compiler transformations
 */
type PrintedCompilerPipelineValue =
  | {
      kind: 'hir';
      name: string;
      fnName: string | null;
      value: string;
    }
  | {
      kind: 'reactive';
      name: string;
      fnName: string | null;
      value: string;
    }
  | {
      kind: 'debug';
      name: string;
      fnName: string | null;
      value: string;
    };
```

**Pipeline Pass Types**:

- **HIR (High-level Intermediate Representation)**:
  - Kind: `'hir'`
  - Represents the high-level IR before reactive transformation
  - Pass name: `'PropagateScopeDependenciesHIR'`
  - Contains function-level optimizations

- **Reactive Function**:
  - Kind: `'reactive'`
  - Represents the reactive function representation
  - Pass name: `'PruneHoistedContexts'`
  - Contains final reactive transformations

- **Debug**:
  - Kind: `'debug'`
  - Generic debug output from any compiler pass
  - Contains all compiler passes with names when using `@DEBUG` mode

**Field Descriptions**:
- `kind`: Type of compiler pass output
- `name`: Name of the compiler pass (e.g., 'PropagateScopeDependenciesHIR')
- `fnName`: Name of the function being compiled, or null if not applicable
- `value`: String representation of the intermediate form

**Usage Context**: This type is primarily used internally by the MCP `compile` tool when the `passName` parameter is provided (HIR, ReactiveFunction, All, or @DEBUG).

---

## Compiler Options

React Compiler configuration passed to the Babel plugin.

```typescript { .api }
/**
 * React Compiler plugin options
 * Configures compilation behavior and logging
 * Note: This type is from babel-plugin-react-compiler
 */
interface PluginOptions {
  /** Panic threshold for compilation errors ('none' = continue on errors) */
  panicThreshold?: 'none' | 'low' | 'medium' | 'high';

  /** Logger for compiler events and intermediate representations */
  logger?: {
    /**
     * Log intermediate representations during compilation
     * @param result - Compiler pipeline value to log
     */
    debugLogIRs?: (result: CompilerPipelineValue) => void;

    /**
     * Log compiler events (errors, warnings, etc.)
     * @param filename - Source filename
     * @param event - Compiler event details
     */
    logEvent?: (filename: string, event: CompilerEvent) => void;
  };
}
```

**Common Configuration**:

```typescript
const compilerOptions: PluginOptions = {
  panicThreshold: 'none',  // Continue compilation despite errors
  logger: {
    debugLogIRs: (result) => {
      // Log intermediate representations
      console.log('Compiler pass:', result.name);
    },
    logEvent: (filename, event) => {
      // Log compilation events
      if (event.kind === 'CompileError') {
        console.error('Compilation error:', event.detail.reason);
      }
    }
  }
};

await compile({
  text: sourceCode,
  file: 'component.tsx',
  options: compilerOptions
});
```

---

## Compiler Event Types

Events emitted during compilation for logging and error handling.

```typescript { .api }
/**
 * Compiler pipeline value passed to debugLogIRs
 * Note: This type is from babel-plugin-react-compiler
 */
interface CompilerPipelineValue {
  kind: 'ast' | 'hir' | 'reactive' | 'debug';
  name: string;
  value: any;
}

/**
 * Compiler event passed to logEvent
 * Note: This type is from babel-plugin-react-compiler
 */
interface CompilerEvent {
  kind: 'CompileError' | 'CompileSuccess' | string;
  fnLoc?: SourceLocation;
  detail?: {
    reason: string;
    loc?: SourceLocation | symbol | null;
  };
}

/**
 * Source location information for errors
 * Note: This type is from babel-plugin-react-compiler
 */
interface SourceLocation {
  start: { line: number; column: number };
  end: { line: number; column: number };
}
```

**Event Handling Example**:

```typescript
const errors: Array<{ message: string; loc: SourceLocation | null }> = [];

const options: PluginOptions = {
  panicThreshold: 'none',
  logger: {
    logEvent: (_filename, event) => {
      if (event.kind === 'CompileError' && event.detail) {
        const loc = event.detail.loc == null || typeof event.detail.loc === 'symbol'
          ? event.fnLoc
          : event.detail.loc;

        errors.push({
          message: event.detail.reason,
          loc: loc || null
        });
      }
    }
  }
};

await compile({ text: code, file: 'test.tsx', options });

// Check for compilation errors
if (errors.length > 0) {
  errors.forEach(err => {
    console.error(`Error: ${err.message} at ${err.loc?.start.line}:${err.loc?.end.line}`);
  });
}
```

---

## Imported Dependencies

The compiler module relies on external Babel and React Compiler packages.

```typescript { .api }
/**
 * Babel core types and functions
 */
import type * as BabelCore from '@babel/core';
import { parseAsync, transformFromAstAsync } from '@babel/core';

/**
 * React Compiler plugin and types
 */
import BabelPluginReactCompiler, {
  type PluginOptions,
  printReactiveFunctionWithOutlined,
  printFunctionWithOutlined
} from 'babel-plugin-react-compiler/src';

/**
 * Code formatting
 */
import * as prettier from 'prettier';
```

**Key Dependencies**:
- `@babel/core@^7.26.0` - Babel parser and transformer
- `babel-plugin-react-compiler` - React Compiler plugin (internal package)
- `prettier@^3.3.3` - Code formatting
