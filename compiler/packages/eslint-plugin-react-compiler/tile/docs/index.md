# ESLint Plugin React Compiler

ESLint Plugin React Compiler is an ESLint plugin that integrates the React Compiler (formerly React Forget) static analysis capabilities directly into the ESLint workflow. It surfaces problematic React code patterns detected by the compiler, enabling developers to identify and fix optimization issues, violations of React's rules, and other code patterns that prevent the React compiler from effectively optimizing components.

## Package Information

- **Package Name**: eslint-plugin-react-compiler
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: `npm install eslint-plugin-react-compiler --save-dev`
- **Peer Dependencies**: eslint >= 7
- **Node Version**: ^14.17.0 || ^16.0.0 || >= 18.0.0

## Core Imports

The plugin exports an ESLint plugin configuration object containing rules.

```javascript
// In ESLint configuration files
// The plugin is referenced by name after registration
module.exports = {
  plugins: ["react-compiler"],
  rules: {
    "react-compiler/react-compiler": "error"
  }
};
```

For programmatic usage:

```javascript
const eslintPluginReactCompiler = require("eslint-plugin-react-compiler");
// Access the rule
const reactCompilerRule = eslintPluginReactCompiler.rules["react-compiler"];
```

## Basic Usage

### ESLint Configuration (Flat Config)

```javascript
const reactCompilerPlugin = require("eslint-plugin-react-compiler");

module.exports = [
  {
    plugins: {
      "react-compiler": reactCompilerPlugin
    },
    rules: {
      "react-compiler/react-compiler": "error"
    }
  }
];
```

### ESLint Configuration (Legacy .eslintrc)

```json
{
  "plugins": ["react-compiler"],
  "rules": {
    "react-compiler/react-compiler": "error"
  }
}
```

### With Options

```json
{
  "rules": {
    "react-compiler/react-compiler": [
      "error",
      {
        "environment": {
          "enableChangeDetectionForDebugging": "none"
        }
      }
    ]
  }
}
```

## Capabilities

### Plugin Export

The main export is an ESLint plugin configuration object.

```typescript { .api }
/**
 * Main plugin export containing rules
 */
interface ESLintPluginReactCompiler {
  rules: {
    "react-compiler": Rule.RuleModule;
  };
}
```

### React Compiler Rule

The `react-compiler` rule runs the React Compiler's static analysis on React components and hooks, reporting errors and providing fix suggestions through ESLint.

```typescript { .api }
/**
 * ESLint rule that surfaces diagnostics from React Compiler
 */
interface ReactCompilerRule extends Rule.RuleModule {
  meta: {
    type: "problem";
    docs: {
      description: "Surfaces diagnostics from React Forget";
      recommended: true;
    };
    fixable: "code";
    hasSuggestions: true;
    schema: [{ type: "object"; additionalProperties: true }];
  };
  create(context: Rule.RuleContext): {};
}
```

The rule's `create` function:
- Parses source code using Babel parser (for TypeScript) or Hermes parser (for JavaScript)
- Runs the React Compiler via `babel-plugin-react-compiler`
- Reports compilation errors via ESLint's context.report()
- Provides fix suggestions for certain errors
- Respects Flow suppression comments (`$FlowFixMe[react-rule-hook]`)

### Rule Options

The rule accepts an options object passed as the second element in the rule configuration array.

```typescript { .api }
/**
 * Configuration options for the react-compiler rule
 */
interface ReactCompilerRuleOptions {
  /**
   * Set of error severity levels that should be reported to ESLint
   * Default: Set([ErrorSeverity.InvalidReact, ErrorSeverity.InvalidJS])
   */
  reportableLevels?: Set<ErrorSeverity>;

  /**
   * Experimental setting to report all compilation bailouts on the compilation unit
   * (e.g. function or hook) instead of the offensive line.
   * Intended for codebases 100% reliant on the compiler for memoization.
   * Default: false
   */
  __unstable_donotuse_reportAllBailouts?: boolean;

  /**
   * Environment configuration for React Compiler
   * This is the PartialEnvironmentConfig type from babel-plugin-react-compiler
   * Validated via validateEnvironmentConfig from babel-plugin-react-compiler
   * See babel-plugin-react-compiler documentation for available configuration options
   */
  environment?: PartialEnvironmentConfig;

  /**
   * Custom logger for logging compilation events
   */
  logger?: Logger;

  /**
   * All other options from babel-plugin-react-compiler's PluginOptions
   * are supported and passed through to the compiler
   */
  [key: string]: any;
}
```

### Error Severity Levels

Error severity levels from `babel-plugin-react-compiler` that can be used with `reportableLevels`.

```typescript { .api }
/**
 * Error severity levels from babel-plugin-react-compiler
 */
enum ErrorSeverity {
  /**
   * Invalid JavaScript syntax or semantically invalid code
   */
  InvalidJS = "InvalidJS",

  /**
   * Code that breaks the rules of React (e.g., conditional hooks)
   */
  InvalidReact = "InvalidReact",

  /**
   * Incorrect configuration of the compiler
   */
  InvalidConfig = "InvalidConfig",

  /**
   * Code that is valid but unsafe to preserve memoization
   */
  CannotPreserveMemoization = "CannotPreserveMemoization",

  /**
   * Unhandled syntax not yet supported by the compiler
   */
  Todo = "Todo",

  /**
   * Internal compiler error indicating critical issues
   */
  Invariant = "Invariant",
}
```

### Logger Interface

Custom logger interface for receiving compilation events.

```typescript { .api }
/**
 * Logger interface from babel-plugin-react-compiler/src/Entrypoint
 */
interface Logger {
  /**
   * Called when a compilation event occurs
   * @param filename - The file being compiled (may be null)
   * @param event - The compilation event with details
   */
  logEvent(filename: string | null, event: LoggerEvent): void;
}

/**
 * Logger event types
 */
type LoggerEvent =
  | {
      kind: "CompileError";
      fnLoc: BabelSourceLocation | null;
      detail: CompilerErrorDetailOptions;
    }
  | {
      kind: "CompileDiagnostic";
      fnLoc: BabelSourceLocation | null;
      detail: Omit<CompilerErrorDetailOptions, "severity" | "suggestions">;
    }
  | {
      kind: "CompileSuccess";
      fnLoc: BabelSourceLocation | null;
      fnName: string | null;
      memoSlots: number;
      memoBlocks: number;
      memoValues: number;
      prunedMemoBlocks: number;
      prunedMemoValues: number;
    }
  | {
      kind: "PipelineError";
      fnLoc: BabelSourceLocation | null;
      data: string;
    };

/**
 * Compiler error detail structure
 */
interface CompilerErrorDetailOptions {
  reason: string;
  description?: string | null;
  severity: ErrorSeverity;
  loc: BabelSourceLocation | null;
  suggestions?: CompilerSuggestion[] | null;
}
```

### Compiler Suggestions

The rule provides automatic fix suggestions for certain compiler errors.

```typescript { .api }
/**
 * Compiler suggestion operation types
 */
enum CompilerSuggestionOperation {
  InsertBefore = 0,
  InsertAfter = 1,
  Remove = 2,
  Replace = 3,
}

/**
 * Compiler suggestion structure (union type)
 */
type CompilerSuggestion =
  | {
      op:
        | CompilerSuggestionOperation.InsertAfter
        | CompilerSuggestionOperation.InsertBefore
        | CompilerSuggestionOperation.Replace;
      range: [number, number];
      description: string;
      text: string;
    }
  | {
      op: CompilerSuggestionOperation.Remove;
      range: [number, number];
      description: string;
    };
```

These suggestions are converted to ESLint's suggestion format and made available to users in their editor or CLI output.

## Supported File Types

The rule automatically selects the appropriate parser based on file extension:

- **TypeScript files** (`.ts`, `.tsx`): Uses `@babel/parser` with TypeScript and JSX plugins
- **JavaScript files** (`.js`, `.jsx`): Uses `hermes-parser` with experimental component syntax support

Both parsers are configured to:
- Parse in module mode
- Support JSX syntax
- Enable experimental React component syntax
- Parse with Babel-compatible AST format

## Error Reporting Behavior

### Default Behavior

By default, the rule reports errors with severity levels:
- `ErrorSeverity.InvalidReact`: Violations of React rules (e.g., conditional hooks)
- `ErrorSeverity.InvalidJS`: Invalid JavaScript patterns that prevent compilation

### Flow Suppression

The rule respects Flow suppression comments. If a line has a `$FlowFixMe[react-rule-hook]` comment on the preceding line, the error will not be reported by ESLint (assuming Flow already caught it).

```javascript
function useHookWithHook() {
  if (cond) {
    // $FlowFixMe[react-rule-hook]
    useConditionalHook(); // Error suppressed
  }
}
```

### Experimental Bailout Reporting

When `__unstable_donotuse_reportAllBailouts` is enabled, all compilation bailouts are reported on the first line of the compilation unit (function or hook) with a message indicating the location of the actual issue.

```javascript
// With __unstable_donotuse_reportAllBailouts: true
function MyComponent() {  // Error reported here
  if (cond) {
    useHook(); // Actual issue at line X:Y
  }
}
// Error message: [ReactCompilerBailout] <reason> (@:X:Y)
```

## Integration with React Compiler

The plugin integrates with `babel-plugin-react-compiler` by:

1. **Parsing**: Converting source code to Babel AST
2. **Transforming**: Running the React Compiler via Babel transform with `noEmit: true` and `panicThreshold: 'none'`
3. **Logging**: Capturing compilation errors via a custom logger injected into the compiler options
4. **Reporting**: Converting compiler errors to ESLint reports with suggestions

The compiler runs in analysis-only mode (`noEmit: true`), meaning it checks the code but doesn't modify it through the ESLint rule itself. Modifications are only applied when users accept fix suggestions.

## Common Usage Patterns

### Basic Error Reporting

```json
{
  "rules": {
    "react-compiler/react-compiler": "error"
  }
}
```

Reports `InvalidReact` and `InvalidJS` errors as ESLint errors.

### Custom Error Levels

```javascript
const { ErrorSeverity } = require("babel-plugin-react-compiler/src");

module.exports = {
  rules: {
    "react-compiler/react-compiler": [
      "error",
      {
        reportableLevels: new Set([
          ErrorSeverity.InvalidReact,
          ErrorSeverity.InvalidJS
          // Add other severity levels as needed
        ])
      }
    ]
  }
};
```

### With Custom Environment

```json
{
  "rules": {
    "react-compiler/react-compiler": [
      "error",
      {
        "environment": {
          "enableChangeDetectionForDebugging": "none"
        }
      }
    ]
  }
}
```

### With Custom Logger

```javascript
module.exports = {
  rules: {
    "react-compiler/react-compiler": [
      "error",
      {
        logger: {
          logEvent(filename, event) {
            console.log(`[${filename}]`, event);
          }
        }
      }
    ]
  }
};
```

### Bailout Tracking for Performance Debugging

```json
{
  "rules": {
    "react-compiler/react-compiler": [
      "warn",
      {
        "__unstable_donotuse_reportAllBailouts": true
      }
    ]
  }
}
```

Use this configuration when you need compilation success signals for performance debugging in codebases that rely entirely on the React Compiler for memoization.

## Types

### Source Location Type

```typescript { .api }
/**
 * Babel source location type
 * From @babel/types
 */
interface BabelSourceLocation {
  start: {
    line: number;
    column: number;
  };
  end: {
    line: number;
    column: number;
  };
}
```

### Environment Config Type

```typescript { .api }
/**
 * Environment configuration type from babel-plugin-react-compiler
 * This is a complex configuration object with many optional boolean and nullable fields
 * that control various aspects of the React Compiler's behavior.
 *
 * Common options include:
 * - enableChangeDetectionForDebugging: Controls change detection for debugging
 * - validateHooksUsage: Validates hooks follow the rules of React
 * - validateRefAccessDuringRender: Validates ref access patterns
 * - And many more compilation and validation options
 *
 * For the complete list of available options, refer to the babel-plugin-react-compiler
 * documentation or source code (HIR/Environment.ts).
 */
type PartialEnvironmentConfig = Partial<{
  // Configuration options from babel-plugin-react-compiler
  // This type accepts any valid EnvironmentConfig options
  [key: string]: any;
}>;
```

### ESLint Rule Context

The rule receives an ESLint Rule.RuleContext with the following relevant properties:

```typescript { .api }
/**
 * ESLint Rule Context (from 'eslint' package)
 */
interface RuleContext {
  /**
   * Source code object (ESLint 8.0+)
   */
  sourceCode?: {
    text: string;
  };

  /**
   * Get source code (ESLint < 8.0)
   */
  getSourceCode(): {
    text: string;
    getAllComments(): Array<{ value: string; loc: BabelSourceLocation }>;
  };

  /**
   * Filename being linted (ESLint 8.0+)
   */
  filename?: string;

  /**
   * Get filename (ESLint < 8.0)
   */
  getFilename(): string;

  /**
   * Rule options array
   */
  options: any[];

  /**
   * Report an error or warning
   */
  report(descriptor: {
    message: string;
    loc: BabelSourceLocation;
    suggest?: Array<{
      desc: string;
      fix(fixer: RuleFixer): any;
    }>;
  }): void;
}

/**
 * ESLint Rule Fixer
 */
interface RuleFixer {
  insertTextBeforeRange(range: [number, number], text: string): any;
  insertTextAfterRange(range: [number, number], text: string): any;
  replaceTextRange(range: [number, number], text: string): any;
  removeRange(range: [number, number]): any;
}
```

## Dependencies

The plugin has the following runtime dependencies:

- **@babel/core**: Babel transformation engine
- **@babel/parser**: Babel JavaScript/TypeScript parser
- **@babel/plugin-proposal-private-methods**: Babel plugin for private methods
- **hermes-parser**: Hermes JavaScript parser
- **zod**: Schema validation library
- **zod-validation-error**: Zod error formatting
- **babel-plugin-react-compiler**: The React Compiler Babel plugin (peer or bundled dependency)

The plugin internally uses `@babel/parser` for TypeScript files and `hermes-parser` for JavaScript files, with `babel-plugin-react-compiler` providing the actual compilation and error detection logic.
