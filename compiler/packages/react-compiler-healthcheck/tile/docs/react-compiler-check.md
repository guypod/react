# React Compiler Check

Tests React components and functions for compatibility with the React Compiler by attempting compilation with babel-plugin-react-compiler and categorizing results.

## Capabilities

### Check Module Interface

The React Compiler check module exports an object with `run()` and `report()` methods.

```typescript { .api }
/**
 * Default export from 'react-compiler-healthcheck/src/checks/reactCompiler'
 */
interface ReactCompilerCheckModule {
  /**
   * Analyzes a source file for React Compiler compatibility
   * @param source - The file content as a string
   * @param path - The file path (must have .js, .ts, .jsx, or .tsx extension)
   */
  run(source: string, path: string): void;

  /**
   * Outputs compilation results to console
   * Shows: "Successfully compiled X out of Y components."
   */
  report(): void;
}
```

### run() Method

Compiles JavaScript and TypeScript files using babel-plugin-react-compiler to test compatibility.

```typescript { .api }
/**
 * Tests React Compiler compatibility for a source file
 *
 * @param source - File content as a string
 * @param path - File path (must match /(js|ts|jsx|tsx)$/)
 * @returns void
 *
 * Side effects:
 * - Parses source with @babel/parser (TypeScript + JSX plugins)
 * - Transforms AST with babel-plugin-react-compiler
 * - Records results in internal arrays for reporting
 */
function run(source: string, path: string): void;
```

**Behavior:**

1. **File Filtering**: Only processes files with `.js`, `.ts`, `.jsx`, or `.tsx` extensions
2. **Parsing**: Uses `@babel/parser` with TypeScript and JSX plugins enabled
3. **Compilation**: Transforms AST using `babel-plugin-react-compiler` with these options:
   - `noEmit: true` - Does not emit transformed code
   - `compilationMode: 'infer'` - Automatically infers component boundaries
   - `panicThreshold: 'critical_errors'` - Only panics on critical errors
4. **Result Tracking**: Categorizes compilation results into three groups:
   - **Successful compilations**: Components that compile without errors
   - **Actionable failures**: Errors with `InvalidReact` or `InvalidJS` severity
   - **Other failures**: Errors with `InvalidConfig`, `Invariant`, `CannotPreserveMemoization`, or `Todo` severity

**Error Categorization:**

Actionable errors (fixable by developers):
- `ErrorSeverity.InvalidReact` - Violations of React rules (hooks, state mutations, etc.)
- `ErrorSeverity.InvalidJS` - Invalid JavaScript patterns

Non-actionable errors:
- `ErrorSeverity.InvalidConfig` - Configuration issues with the compiler
- `ErrorSeverity.Invariant` - Internal compiler invariant violations
- `ErrorSeverity.CannotPreserveMemoization` - Memoization cannot be preserved
- `ErrorSeverity.Todo` - Unimplemented compiler features

**Deduplication:**

The check deduplicates error events by source location (`filename:startPos:endPos`) to avoid counting the same component multiple times when multiple error events are emitted for a single function.

**Usage Example:**

```typescript
import reactCompilerCheck from 'react-compiler-healthcheck/src/checks/reactCompiler';

const componentSource = `
export function MyComponent() {
  const [count, setCount] = useState(0);
  return <div onClick={() => setCount(count + 1)}>{count}</div>;
}
`;

reactCompilerCheck.run(componentSource, 'MyComponent.tsx');
```

### report() Method

Outputs compilation success statistics to the console.

```typescript { .api }
/**
 * Outputs compilation results to console
 * Format: "Successfully compiled X out of Y components."
 * Color: Green text
 * @returns void
 */
function report(): void;
```

**Output Format:**

```
Successfully compiled X out of Y components.
```

Where:
- **X**: Number of components that compiled successfully
- **Y**: Total unique components analyzed (successful + failed)

**Calculation:**

- Successful count: Number of `CompileSuccess` events
- Total count: Successful + unique failure locations
- Failures are deduplicated by function source location to avoid double-counting

**Usage Example:**

```typescript
import reactCompilerCheck from 'react-compiler-healthcheck/src/checks/reactCompiler';

// After running checks on multiple files
reactCompilerCheck.report();
// Output: "Successfully compiled 42 out of 45 components."
```

### Complete Programmatic Usage

```typescript
import reactCompilerCheck from 'react-compiler-healthcheck/src/checks/reactCompiler';
import * as fs from 'fs/promises';
import { glob } from 'fast-glob';

async function checkProject() {
  const files = await glob('src/**/*.{ts,tsx}', {
    ignore: ['**/node_modules/**', '**/__tests__/**']
  });

  for (const file of files) {
    const source = await fs.readFile(file, 'utf-8');
    reactCompilerCheck.run(source, file);
  }

  reactCompilerCheck.report();
}

checkProject();
```

## Types

### LoggerEvent (Internal)

Used internally to track compilation events. Not exported from the module.

```typescript { .api }
/**
 * Internal type used for tracking compilation events
 * Combines RawLoggerEvent from babel-plugin-react-compiler with filename
 */
type LoggerEvent = {
  kind: 'CompileSuccess' | 'CompileError' | 'CompileDiagnostic' | 'PipelineError';
  filename: string | null;
  fnLoc?: {
    start: number;
    end: number;
  };
  detail?: CompilerErrorDetailOptions;
};
```

### CompilerErrorDetailOptions

Imported from babel-plugin-react-compiler, contains error severity and details.

```typescript { .api }
/**
 * Error detail options from babel-plugin-react-compiler
 */
interface CompilerErrorDetailOptions {
  severity: ErrorSeverity;
  // Additional fields from babel-plugin-react-compiler
}

enum ErrorSeverity {
  InvalidReact = 'InvalidReact',
  InvalidJS = 'InvalidJS',
  InvalidConfig = 'InvalidConfig',
  Invariant = 'Invariant',
  CannotPreserveMemoization = 'CannotPreserveMemoization',
  Todo = 'Todo'
}
```

## Dependencies

This check module requires:
- **@babel/core**: AST transformation engine
- **@babel/parser**: JavaScript/TypeScript parser
- **babel-plugin-react-compiler**: React Compiler Babel plugin (internal dependency from monorepo)
- **chalk**: Terminal string styling for colored output

## Notes

- The check runs in "infer" mode, automatically detecting component and hook boundaries
- Compilation is non-destructive (`noEmit: true`) - original code is never modified
- The check handles both TypeScript and Flow syntax but defaults to TypeScript parsing
- Multiple error events for the same function are deduplicated by source location
- The module maintains internal state across `run()` calls, accumulating results until `report()` is called
