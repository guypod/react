# Error Handling

Comprehensive error handling system with detailed diagnostics, severity levels, and suggestions for fixing compilation issues.

## Capabilities

### CompilerError

Main error class for React Compiler errors. Accumulates multiple error details and provides methods for error management.

```typescript { .api }
class CompilerError extends Error {
  /** Collection of error details */
  details: Array<CompilerErrorDetail>;

  /** Always 'ReactCompilerError' */
  name: string;

  /** Formatted error message */
  message: string;

  constructor(...args: Array<any>);

  /**
   * Add an error detail
   * @param options - Error detail options
   * @returns The created error detail
   */
  push(options: CompilerErrorDetailOptions): CompilerErrorDetail;

  /**
   * Add a pre-constructed error detail
   * @param detail - Pre-constructed error detail
   * @returns The error detail
   */
  pushErrorDetail(detail: CompilerErrorDetail): CompilerErrorDetail;

  /**
   * Check if any errors exist
   * @returns true if errors exist
   */
  hasErrors(): boolean;

  /**
   * Check if any error is critical
   * Critical errors: Invariant, InvalidJS, InvalidReact, InvalidConfig
   * @returns true if any critical error exists
   */
  isCritical(): boolean;

  /**
   * Get formatted error message
   * @returns Formatted error string
   */
  toString(): string;

  /**
   * Assert a condition or throw an invariant error
   * @param condition - Condition to assert
   * @param options - Error options (severity is always Invariant)
   * @throws CompilerError if condition is false
   */
  static invariant(
    condition: unknown,
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): asserts condition;

  /**
   * Throw a Todo error for unhandled syntax/features
   * @param options - Error options (severity is always Todo)
   * @throws CompilerError
   */
  static throwTodo(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;

  /**
   * Throw an InvalidJS error for invalid JavaScript
   * @param options - Error options (severity is always InvalidJS)
   * @throws CompilerError
   */
  static throwInvalidJS(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;

  /**
   * Throw an InvalidReact error for code breaking React rules
   * @param options - Error options (severity is always InvalidReact)
   * @throws CompilerError
   */
  static throwInvalidReact(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;

  /**
   * Throw an InvalidConfig error for incorrect configuration
   * @param options - Error options (severity is always InvalidConfig)
   * @throws CompilerError
   */
  static throwInvalidConfig(
    options: Omit<CompilerErrorDetailOptions, "severity">
  ): never;

  /**
   * Throw an error with custom severity
   * @param options - Error options with severity
   * @throws CompilerError
   */
  static throw(options: CompilerErrorDetailOptions): never;
}
```

**Usage Examples:**

```typescript
import { CompilerError, ErrorSeverity } from "babel-plugin-react-compiler";

// Catching compilation errors
try {
  compile(/* ... */);
} catch (err) {
  if (err instanceof CompilerError) {
    console.error("Compilation failed:", err.toString());

    // Check error details
    for (const detail of err.details) {
      console.log(`  ${detail.severity}: ${detail.reason}`);
      if (detail.loc) {
        console.log(
          `    at line ${detail.loc.start.line}, column ${detail.loc.start.column}`
        );
      }
    }

    // Check if critical
    if (err.isCritical()) {
      console.error("Critical error - compilation cannot proceed");
    }
  }
}

// Creating errors programmatically
const error = new CompilerError();

error.push({
  reason: "Invalid hook usage",
  description: "Hooks must be called at the top level",
  severity: ErrorSeverity.InvalidReact,
  loc: { start: { line: 10, column: 5 }, end: { line: 10, column: 20 } },
  suggestions: [
    {
      op: "Replace",
      range: [10, 20],
      text: "const value = useMemo(() => compute(), []);",
      description: "Move hook to top level",
    },
  ],
});

if (error.hasErrors()) {
  throw error;
}

// Using static methods
function validateAST(node: t.Node) {
  CompilerError.invariant(node.type === "FunctionDeclaration", {
    reason: "Expected function declaration",
    description: `Got ${node.type} instead`,
    loc: node.loc,
  });
}

function handleUnimplemented(feature: string, loc: SourceLocation) {
  CompilerError.throwTodo({
    reason: `${feature} is not yet supported`,
    description: "This feature is planned for a future release",
    loc,
  });
}

function validateReactRules(node: t.Node) {
  if (isInvalidHookCall(node)) {
    CompilerError.throwInvalidReact({
      reason: "Hooks cannot be called conditionally",
      description: "Hooks must be called in the same order on every render",
      loc: node.loc,
    });
  }
}
```

---

### CompilerErrorDetail

Individual error detail with location, severity, and optional suggestions.

```typescript { .api }
class CompilerErrorDetail {
  constructor(options: CompilerErrorDetailOptions);

  /** Error reason */
  get reason(): string;

  /** Additional description */
  get description(): string | null | undefined;

  /** Error severity level */
  get severity(): ErrorSeverity;

  /** Source location */
  get loc(): SourceLocation | null;

  /** Suggested fixes */
  get suggestions(): Array<CompilerSuggestion> | null | undefined;

  /**
   * Format the error message
   * @returns Formatted error string
   */
  printErrorMessage(): string;

  /**
   * Same as printErrorMessage()
   * @returns Formatted error string
   */
  toString(): string;
}
```

**Usage Example:**

```typescript
import { CompilerErrorDetail, ErrorSeverity } from "babel-plugin-react-compiler";

const detail = new CompilerErrorDetail({
  reason: "Cannot preserve memoization",
  description: "The value is mutated after memoization",
  severity: ErrorSeverity.CannotPreserveMemoization,
  loc: {
    start: { line: 15, column: 10 },
    end: { line: 15, column: 25 },
  },
  suggestions: [
    {
      op: "Replace",
      range: [15, 25],
      text: "const memoized = useMemo(() => value, []);",
      description: "Wrap in useMemo to preserve memoization",
    },
  ],
});

console.log(detail.reason); // "Cannot preserve memoization"
console.log(detail.severity); // ErrorSeverity.CannotPreserveMemoization
console.log(detail.printErrorMessage()); // Formatted error message
```

---

### ErrorSeverity

Enum defining error severity levels for compiler diagnostics.

```typescript { .api }
enum ErrorSeverity {
  /**
   * Invalid JavaScript syntax or semantics
   * Cannot be compiled due to language-level issues
   */
  InvalidJS = "InvalidJS",

  /**
   * Code breaks React rules
   * Violates Rules of Hooks or other React requirements
   */
  InvalidReact = "InvalidReact",

  /**
   * Incorrect compiler configuration
   * Invalid plugin options or environment config
   */
  InvalidConfig = "InvalidConfig",

  /**
   * Unsafe to preserve existing memoization
   * Manual memoization cannot be guaranteed to work correctly
   */
  CannotPreserveMemoization = "CannotPreserveMemoization",

  /**
   * Unhandled syntax or feature
   * Feature not yet implemented in the compiler
   */
  Todo = "Todo",

  /**
   * Internal compiler error
   * Bug in the compiler itself
   */
  Invariant = "Invariant",
}
```

**Usage Example:**

```typescript
import { ErrorSeverity, CompilerError } from "babel-plugin-react-compiler";

function categorizeError(severity: ErrorSeverity): string {
  switch (severity) {
    case ErrorSeverity.InvalidJS:
      return "JavaScript Error";
    case ErrorSeverity.InvalidReact:
      return "React Rules Violation";
    case ErrorSeverity.InvalidConfig:
      return "Configuration Error";
    case ErrorSeverity.CannotPreserveMemoization:
      return "Memoization Warning";
    case ErrorSeverity.Todo:
      return "Not Implemented";
    case ErrorSeverity.Invariant:
      return "Internal Error";
  }
}

// Check if error is critical
function isCritical(severity: ErrorSeverity): boolean {
  return (
    severity === ErrorSeverity.Invariant ||
    severity === ErrorSeverity.InvalidJS ||
    severity === ErrorSeverity.InvalidReact ||
    severity === ErrorSeverity.InvalidConfig
  );
}
```

---

### CompilerSuggestionOperation

Enum defining operations for compiler error suggestions.

```typescript { .api }
enum CompilerSuggestionOperation {
  /** Insert code before the location */
  InsertBefore,

  /** Insert code after the location */
  InsertAfter,

  /** Remove code at the location */
  Remove,

  /** Replace code at the location */
  Replace,
}
```

**Usage in Suggestions:**

```typescript
import {
  CompilerError,
  CompilerSuggestionOperation,
  ErrorSeverity,
} from "babel-plugin-react-compiler";

const error = new CompilerError();

error.push({
  reason: "Hook called conditionally",
  description: "Hooks must be called unconditionally at the top level",
  severity: ErrorSeverity.InvalidReact,
  loc: { start: { line: 10, column: 5 }, end: { line: 10, column: 20 } },
  suggestions: [
    {
      op: CompilerSuggestionOperation.Replace,
      range: [10, 20],
      text: "const value = condition ? useMemo(() => compute(), []) : null;",
      description: "Move condition outside hook call",
    },
    {
      op: CompilerSuggestionOperation.Remove,
      range: [10, 20],
      text: "",
      description: "Remove conditional hook call",
    },
  ],
});
```

---

### CompilerErrorDetailOptions

Type for creating compiler error details.

```typescript { .api }
type CompilerErrorDetailOptions = {
  /** Error reason (brief description) */
  reason: string;

  /** Additional description (optional) */
  description?: string | null | undefined;

  /** Error severity level */
  severity: ErrorSeverity;

  /** Source location where error occurred */
  loc: SourceLocation | null;

  /** Suggested fixes (optional) */
  suggestions?: Array<CompilerSuggestion> | null | undefined;
};
```

---

### CompilerSuggestion

Type for error fix suggestions.

```typescript { .api }
type CompilerSuggestion = {
  /** Operation type */
  op: CompilerSuggestionOperation;

  /** Source range [start, end] */
  range: [number, number];

  /** Replacement or insertion text */
  text: string;

  /** Description of the suggestion */
  description: string;
};
```

**Complete Example:**

```typescript
import {
  CompilerError,
  ErrorSeverity,
  CompilerSuggestionOperation,
  runBabelPluginReactCompiler,
} from "babel-plugin-react-compiler";

function compileWithErrorHandling(code: string, filename: string) {
  try {
    const result = runBabelPluginReactCompiler(
      code,
      filename,
      "typescript",
      { compilationMode: "infer" }
    );
    return { success: true, code: result.code };
  } catch (err) {
    if (err instanceof CompilerError) {
      const errors = [];

      for (const detail of err.details) {
        const error = {
          file: filename,
          line: detail.loc?.start.line,
          column: detail.loc?.start.column,
          severity: detail.severity,
          reason: detail.reason,
          description: detail.description,
          suggestions: detail.suggestions?.map((s) => ({
            operation: s.op,
            text: s.text,
            description: s.description,
          })),
        };

        errors.push(error);

        // Log based on severity
        if (
          detail.severity === ErrorSeverity.Invariant ||
          detail.severity === ErrorSeverity.InvalidJS
        ) {
          console.error("CRITICAL:", error.reason);
        } else if (detail.severity === ErrorSeverity.Todo) {
          console.warn("NOT IMPLEMENTED:", error.reason);
        } else {
          console.log("WARNING:", error.reason);
        }
      }

      return { success: false, errors };
    }

    throw err;
  }
}

// Usage
const result = compileWithErrorHandling(
  `
function MyComponent({ items }) {
  if (items.length > 0) {
    const data = useMemo(() => process(items), [items]); // Potentially invalid
  }
  return <div />;
}
`,
  "MyComponent.tsx"
);

if (result.success) {
  console.log("Compiled successfully:", result.code);
} else {
  console.log("Compilation failed with errors:", result.errors);
}
```
