# Error Handling

Comprehensive error system for React Compiler with detailed diagnostics, source locations, and fix suggestions.

## Capabilities

### CompilerError Class

Main error class for compilation errors with detailed error information.

```typescript { .api }
/**
 * Main error class for compilation errors
 * Extends Error with multiple error details and helper methods
 */
class CompilerError extends Error {
  /**
   * Array of error details
   * Can contain multiple errors from a single compilation
   */
  details: Array<CompilerErrorDetail>;

  /**
   * Assert invariant condition
   * @param condition - Condition to assert
   * @param options - Error options (severity automatically set to Invariant)
   * @throws CompilerError if condition is falsy
   */
  static invariant(
    condition: unknown,
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): asserts condition;

  /**
   * Throw TODO error for unimplemented features
   * @param options - Error options (severity automatically set to Todo)
   * @throws CompilerError with Todo severity
   */
  static throwTodo(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;

  /**
   * Throw invalid JavaScript error
   * @param options - Error options (severity automatically set to InvalidJS)
   * @throws CompilerError with InvalidJS severity
   */
  static throwInvalidJS(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;

  /**
   * Throw invalid React error
   * @param options - Error options (severity automatically set to InvalidReact)
   * @throws CompilerError with InvalidReact severity
   */
  static throwInvalidReact(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;

  /**
   * Throw invalid configuration error
   * @param options - Error options (severity automatically set to InvalidConfig)
   * @throws CompilerError with InvalidConfig severity
   */
  static throwInvalidConfig(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;

  /**
   * Throw generic error with specified severity
   * @param options - Error options with severity
   * @throws CompilerError
   */
  static throw(options: CompilerErrorDetailOptions): never;

  /**
   * Add error detail to existing error
   * @param options - Error detail options
   * @returns Added CompilerErrorDetail
   */
  push(options: CompilerErrorDetailOptions): CompilerErrorDetail;

  /**
   * Add error detail object to existing error
   * @param detail - Pre-constructed error detail
   * @returns Added CompilerErrorDetail
   */
  pushErrorDetail(detail: CompilerErrorDetail): CompilerErrorDetail;

  /**
   * Check if error contains any error-level details
   * @returns True if has errors (not just warnings/info)
   */
  hasErrors(): boolean;

  /**
   * Check if error is critical
   * Critical errors prevent code generation
   * @returns True if contains critical errors
   */
  isCritical(): boolean;

  /**
   * Convert to string representation
   * @returns Formatted error message with all details
   */
  toString(): string;
}
```

**Usage Examples:**

```typescript
import { CompilerError } from "babel-plugin-react-compiler";

// Assert invariant
function processNode(node) {
  CompilerError.invariant(node !== null, {
    reason: "Node cannot be null",
    description: "Expected valid AST node",
    loc: node?.loc || null,
  });
}

// Throw TODO for unimplemented
function handleExperimentalFeature() {
  CompilerError.throwTodo({
    reason: "Experimental feature not yet implemented",
    description: "Support for do-expressions is coming in future release",
    loc: node.loc,
  });
}

// Throw invalid React error
function validateHookCall(path) {
  if (isConditional(path)) {
    CompilerError.throwInvalidReact({
      reason: "Hooks cannot be called conditionally",
      description: "Hooks must be called in the same order on every render",
      loc: path.node.loc,
      suggestions: [
        {
          op: CompilerSuggestionOperation.Replace,
          range: [path.node.start, path.node.end],
          description: "Move hook to top level of component",
          text: "// Move this hook call to component top level",
        },
      ],
    });
  }
}

// Accumulate multiple errors
try {
  const errors = new CompilerError();

  for (const node of nodes) {
    try {
      validate(node);
    } catch (err) {
      if (err instanceof CompilerError) {
        err.details.forEach((detail) => errors.pushErrorDetail(detail));
      }
    }
  }

  if (errors.hasErrors()) {
    throw errors;
  }
} catch (error) {
  console.error(error.toString());
}
```

### CompilerErrorDetail Class

Individual error detail with location and suggestions.

```typescript { .api }
/**
 * Individual error detail with location and suggestions
 */
class CompilerErrorDetail {
  /**
   * Error detail options
   */
  options: CompilerErrorDetailOptions;

  /**
   * Get error reason (short message)
   */
  get reason(): string;

  /**
   * Get error description (detailed message)
   */
  get description(): string | null | undefined;

  /**
   * Get error severity
   */
  get severity(): ErrorSeverity;

  /**
   * Get source location
   */
  get loc(): SourceLocation | null;

  /**
   * Get fix suggestions
   */
  get suggestions(): Array<CompilerSuggestion> | null | undefined;

  /**
   * Print formatted error message
   * @returns Formatted error string
   */
  printErrorMessage(): string;

  /**
   * Convert to string representation
   * @returns Formatted error detail
   */
  toString(): string;
}
```

### ErrorSeverity Enum

Error severity levels for categorizing errors.

```typescript { .api }
/**
 * Error severity levels
 */
enum ErrorSeverity {
  /**
   * Invalid JavaScript syntax or semantics
   * Code violates JavaScript language rules
   */
  InvalidJS = "InvalidJS",

  /**
   * Invalid React usage
   * Code violates Rules of React
   */
  InvalidReact = "InvalidReact",

  /**
   * Invalid configuration
   * Plugin or environment config is incorrect
   */
  InvalidConfig = "InvalidConfig",

  /**
   * Cannot preserve memoization
   * Existing memoization guarantees cannot be maintained
   */
  CannotPreserveMemoization = "CannotPreserveMemoization",

  /**
   * TODO/Unimplemented feature
   * Feature not yet supported by compiler
   */
  Todo = "Todo",

  /**
   * Invariant violation
   * Internal compiler assertion failed
   */
  Invariant = "Invariant",
}
```

### CompilerSuggestionOperation Enum

Operations for error fix suggestions.

```typescript { .api }
/**
 * Operations for error fix suggestions
 * Note: This is a numeric enum (0, 1, 2, 3)
 */
enum CompilerSuggestionOperation {
  /**
   * Insert text before range (0)
   */
  InsertBefore,

  /**
   * Insert text after range (1)
   */
  InsertAfter,

  /**
   * Remove range (2)
   */
  Remove,

  /**
   * Replace range with text (3)
   */
  Replace,
}
```

## Types

### CompilerErrorDetailOptions

Options for creating error details.

```typescript { .api }
interface CompilerErrorDetailOptions {
  /**
   * Short error message (required)
   */
  reason: string;

  /**
   * Detailed error explanation (optional)
   */
  description?: string | null;

  /**
   * Error severity
   */
  severity: ErrorSeverity;

  /**
   * Source location where error occurred (optional)
   */
  loc?: SourceLocation | null;

  /**
   * Fix suggestions (optional)
   */
  suggestions?: Array<CompilerSuggestion> | null;
}
```

### CompilerSuggestion

Suggestion for fixing an error.

```typescript { .api }
type CompilerSuggestion =
  | {
      /**
       * Insert, replace operations with text
       */
      op:
        | CompilerSuggestionOperation.InsertBefore
        | CompilerSuggestionOperation.InsertAfter
        | CompilerSuggestionOperation.Replace;

      /**
       * Source range [start, end] in characters
       */
      range: [number, number];

      /**
       * Description of the fix
       */
      description: string;

      /**
       * Text to insert or replace with
       */
      text: string;
    }
  | {
      /**
       * Remove operation
       */
      op: CompilerSuggestionOperation.Remove;

      /**
       * Source range [start, end] in characters
       */
      range: [number, number];

      /**
       * Description of the fix
       */
      description: string;
    };
```

### SourceLocation

Source code location information.

```typescript { .api }
interface SourceLocation {
  /**
   * Start position
   */
  start: {
    line: number; // 1-indexed
    column: number; // 0-indexed
  };

  /**
   * End position
   */
  end: {
    line: number; // 1-indexed
    column: number; // 0-indexed
  };

  /**
   * Source filename (optional)
   */
  filename?: string | null;
}
```

## Error Handling Patterns

### Try-Catch with Error Classification

```typescript
import {
  runBabelPluginReactCompiler,
  CompilerError,
  ErrorSeverity,
} from "babel-plugin-react-compiler";

try {
  const result = runBabelPluginReactCompiler(
    sourceCode,
    "MyComponent.tsx",
    "typescript",
    options
  );

  console.log("✓ Compilation successful");
  return result;
} catch (error) {
  if (error instanceof CompilerError) {
    const criticalErrors = error.details.filter(
      (d) =>
        d.severity === ErrorSeverity.InvalidJS ||
        d.severity === ErrorSeverity.InvalidReact
    );

    const todos = error.details.filter((d) => d.severity === ErrorSeverity.Todo);

    if (criticalErrors.length > 0) {
      console.error("✗ Critical compilation errors:");
      criticalErrors.forEach((detail) => {
        console.error(`  ${detail.reason}`);
        if (detail.loc) {
          console.error(
            `    at line ${detail.loc.start.line}, column ${detail.loc.start.column}`
          );
        }
      });
    }

    if (todos.length > 0) {
      console.warn("⚠ Unimplemented features:");
      todos.forEach((detail) => console.warn(`  ${detail.reason}`));
    }

    return null;
  }

  throw error; // Re-throw non-compiler errors
}
```

### Error Detail Inspection

```typescript
import {
  CompilerError,
  ErrorSeverity,
  CompilerSuggestionOperation,
} from "babel-plugin-react-compiler";

function handleCompilerError(error: CompilerError) {
  console.error(`Compilation failed with ${error.details.length} error(s):\n`);

  error.details.forEach((detail, index) => {
    console.error(`Error ${index + 1}:`);
    console.error(`  Severity: ${detail.severity}`);
    console.error(`  Reason: ${detail.reason}`);

    if (detail.description) {
      console.error(`  Description: ${detail.description}`);
    }

    if (detail.loc) {
      const { start, end, filename } = detail.loc;
      console.error(`  Location: ${filename || "unknown"}:${start.line}:${start.column}`);
      console.error(`    Range: ${start.line}:${start.column} to ${end.line}:${end.column}`);
    }

    if (detail.suggestions && detail.suggestions.length > 0) {
      console.error(`  Suggestions:`);
      detail.suggestions.forEach((suggestion, i) => {
        console.error(`    ${i + 1}. ${suggestion.description}`);
        console.error(`       Operation: ${suggestion.op}`);
        console.error(`       Range: [${suggestion.range[0]}, ${suggestion.range[1]}]`);

        if (suggestion.op !== CompilerSuggestionOperation.Remove) {
          console.error(`       Text: ${suggestion.text}`);
        }
      });
    }

    console.error("");
  });
}
```

### Applying Fix Suggestions

```typescript
import {
  CompilerError,
  CompilerSuggestionOperation,
  type CompilerSuggestion,
} from "babel-plugin-react-compiler";

function applyFix(sourceCode: string, suggestion: CompilerSuggestion): string {
  const [start, end] = suggestion.range;

  switch (suggestion.op) {
    case CompilerSuggestionOperation.InsertBefore:
      return sourceCode.slice(0, start) + suggestion.text + sourceCode.slice(start);

    case CompilerSuggestionOperation.InsertAfter:
      return sourceCode.slice(0, end) + suggestion.text + sourceCode.slice(end);

    case CompilerSuggestionOperation.Replace:
      return sourceCode.slice(0, start) + suggestion.text + sourceCode.slice(end);

    case CompilerSuggestionOperation.Remove:
      return sourceCode.slice(0, start) + sourceCode.slice(end);
  }
}

// Apply all suggestions from first error
try {
  compile(sourceCode);
} catch (error) {
  if (error instanceof CompilerError && error.details[0]?.suggestions) {
    const suggestions = error.details[0].suggestions;
    let fixedCode = sourceCode;

    // Apply suggestions in reverse order to maintain offsets
    suggestions
      .slice()
      .sort((a, b) => b.range[0] - a.range[0])
      .forEach((suggestion) => {
        fixedCode = applyFix(fixedCode, suggestion);
      });

    console.log("Applied fixes:");
    console.log(fixedCode);
  }
}
```

### Custom Error Handling in Build Tools

```typescript
import {
  runBabelPluginReactCompiler,
  CompilerError,
  ErrorSeverity,
} from "babel-plugin-react-compiler";

interface CompilationResult {
  success: boolean;
  code: string | null;
  errors: Array<{
    severity: string;
    message: string;
    location?: {
      file: string;
      line: number;
      column: number;
    };
  }>;
}

function compileWithErrorHandling(
  filename: string,
  source: string
): CompilationResult {
  try {
    const result = runBabelPluginReactCompiler(
      source,
      filename,
      "typescript",
      { panicThreshold: "critical_errors" }
    );

    return {
      success: true,
      code: result.code,
      errors: [],
    };
  } catch (error) {
    if (error instanceof CompilerError) {
      return {
        success: false,
        code: null,
        errors: error.details.map((detail) => ({
          severity: detail.severity,
          message: detail.reason,
          location: detail.loc
            ? {
                file: detail.loc.filename || filename,
                line: detail.loc.start.line,
                column: detail.loc.start.column,
              }
            : undefined,
        })),
      };
    }

    // Unexpected error
    return {
      success: false,
      code: null,
      errors: [
        {
          severity: "Error",
          message: error instanceof Error ? error.message : String(error),
        },
      ],
    };
  }
}
```

## Error Severity Guidelines

### InvalidJS
- Syntax errors
- Unsupported JavaScript features
- Invalid AST structures

### InvalidReact
- Hooks called conditionally
- Hooks called in loops
- setState called during render
- Ref accessed during render
- Invalid component patterns

### InvalidConfig
- Invalid plugin options
- Invalid environment config
- Missing required configuration

### CannotPreserveMemoization
- Existing useMemo cannot be preserved
- Manual memoization conflicts with compiler
- Memoization guarantees cannot be maintained

### Todo
- Unimplemented language features
- Unsupported React patterns
- Features planned for future releases

### Invariant
- Internal compiler bugs
- Unexpected state violations
- Should be reported as compiler issues
