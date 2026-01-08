# eslint-plugin-react-compiler

ESLint plugin that surfaces problematic React code patterns found by the React compiler during the linting process, enabling developers to catch compilation issues in their development workflow.

## Package Information

- **Package Name**: eslint-plugin-react-compiler
- **Package Type**: npm
- **Language**: TypeScript/JavaScript
- **Installation**: `npm install eslint-plugin-react-compiler --save-dev`

## Core Imports

This is an ESLint plugin, so it's imported via ESLint configuration rather than direct JavaScript imports.

```json
{
  "plugins": ["react-compiler"]
}
```

## Basic Usage

Add the plugin to your ESLint configuration and enable the rule:

```json
{
  "plugins": ["react-compiler"],
  "rules": {
    "react-compiler/react-compiler": "error"
  }
}
```

The plugin will analyze React components and hooks using the React Compiler and report any issues as ESLint errors.

## Capabilities

### Plugin Export Structure

The plugin exports a standard ESLint plugin object containing rules.

```javascript { .api }
module.exports = {
  rules: {
    "react-compiler": Rule.RuleModule
  }
};
```

### React Compiler Rule

The main and only rule provided by this plugin. It analyzes React code using the React Compiler and reports diagnostics as ESLint errors or warnings.

```typescript { .api }
/**
 * ESLint rule that surfaces React Compiler diagnostics
 */
const rule: Rule.RuleModule = {
  meta: {
    type: "problem",
    docs: {
      description: "Surfaces diagnostics from React Forget",
      recommended: true
    },
    fixable: "code",
    hasSuggestions: true,
    schema: [{ type: "object", additionalProperties: true }]
  },
  create(context: Rule.RuleContext): {}
};
```

**Rule Name**: `react-compiler/react-compiler`

**Rule Type**: `problem`

**Fixable**: Yes (some violations can be auto-fixed)

**Has Suggestions**: Yes (provides fix suggestions for applicable violations)

### Rule Configuration Options

The rule accepts a configuration object with the following options:

```typescript { .api }
interface RuleOptions {
  /**
   * Set of error severity levels that should be reported as ESLint diagnostics
   * Default: Set([ErrorSeverity.InvalidReact, ErrorSeverity.InvalidJS])
   */
  reportableLevels?: Set<ErrorSeverity>;

  /**
   * Experimental option to report all compilation bailouts at the function/hook level
   * instead of the specific line. Intended for codebases 100% reliant on the compiler
   * for memoization. Not recommended for general use.
   * Default: false
   */
  __unstable_donotuse_reportAllBailouts?: boolean;

  /**
   * React Compiler environment configuration
   */
  environment?: PartialEnvironmentConfig | null;

  /**
   * Custom logger for compiler events
   */
  logger?: Logger | null;

  /**
   * Determines the strategy for which functions to compile
   * Default: 'infer'
   */
  compilationMode?: CompilationMode;

  /**
   * Feature gating configuration. When specified, the compiler emits two versions of
   * the function: compiled and uncompiled, gated by importing a flag from the specified module.
   * Default: null
   */
  gating?: ExternalFunction | null;

  /**
   * Error handling strategy for compilation errors
   * - 'all_errors': Panic on any error, skipping rest of file
   * - 'critical_errors': Panic only on critical errors, skip erroring functions otherwise
   * - 'none': Never panic, always continue compilation
   * Default: 'none'
   */
  panicThreshold?: 'all_errors' | 'critical_errors' | 'none';

  /**
   * When enabled, the compiler analyzes and lints code but skips code generation
   * Default: false
   */
  noEmit?: boolean;

  /**
   * Custom module path for importing useMemoCache instead of 'react/compiler-runtime'
   * Example: 'react-compiler-runtime'
   * Default: null
   */
  runtimeModule?: string | null | undefined;

  /**
   * Set of ESLint rule names - code that suppresses these rules will skip compilation
   * Pass empty array to disable this feature and compile even if ESLint rules are suppressed
   * Default: undefined (uses default React ESLint rules)
   */
  eslintSuppressionRules?: Array<string> | null | undefined;

  /**
   * Check for Flow suppression comments to skip compilation
   * Default: false
   */
  flowSuppressions?: boolean;

  /**
   * Ignore "use no forget" annotations (for testing only, not for production)
   * Default: false
   */
  ignoreUseNoForget?: boolean;

  /**
   * Specify which source files to compile
   * Can be an array of glob patterns or a function that returns boolean for a filename
   * Default: null (compile all files)
   */
  sources?: Array<string> | ((filename: string) => boolean) | null;

  /**
   * Enable customized support for react-native-reanimated library
   * Default: true
   */
  enableReanimatedCheck?: boolean;
}
```

**Configuration Examples:**

Basic configuration with error severity:
```json
{
  "rules": {
    "react-compiler/react-compiler": "error"
  }
}
```

With custom reportable levels:
```javascript
// In ESLint config (JS format required for Set constructor)
import { ErrorSeverity } from 'babel-plugin-react-compiler/src';

module.exports = {
  rules: {
    "react-compiler/react-compiler": ["error", {
      reportableLevels: new Set([
        ErrorSeverity.InvalidReact,
        ErrorSeverity.InvalidJS,
        ErrorSeverity.Todo
      ])
    }]
  }
};
```

With experimental bailout reporting:
```javascript
module.exports = {
  rules: {
    "react-compiler/react-compiler": ["error", {
      __unstable_donotuse_reportAllBailouts: true
    }]
  }
};
```

### Error Severity Levels

The plugin uses severity levels from `babel-plugin-react-compiler` to categorize issues:

```typescript { .api }
enum ErrorSeverity {
  /**
   * Invalid JavaScript syntax that cannot be compiled
   */
  InvalidJS,

  /**
   * Invalid React patterns that violate React rules (e.g., hooks rules)
   */
  InvalidReact,

  /**
   * Incorrect compiler configuration
   */
  InvalidConfig,

  /**
   * Unsafe to preserve existing memoization guarantees
   */
  CannotPreserveMemoization,

  /**
   * Unimplemented compiler features or unsupported syntax
   */
  Todo,

  /**
   * Internal compiler error or invariant violation
   */
  Invariant,
}
```

### Logger Interface

Custom logger for handling compiler events:

```typescript { .api }
interface Logger {
  /**
   * Log a compiler event
   * @param filename - The file being processed (null for non-file-specific events)
   * @param event - The compiler event
   */
  logEvent(filename: string | null, event: LoggerEvent): void;
}

/**
 * Union type for all possible compiler events that can be logged
 */
type LoggerEvent = CompileError | CompileDiagnostic | CompileSuccess | PipelineError;

/**
 * Logged when compilation fails for a function
 */
interface CompileError {
  kind: 'CompileError';
  fnLoc: SourceLocation | null;
  detail: CompilerErrorDetailOptions;
}

/**
 * Logged for compilation diagnostics (warnings or info)
 */
interface CompileDiagnostic {
  kind: 'CompileDiagnostic';
  fnLoc: SourceLocation | null;
  detail: CompilerErrorDetailOptions;
}

/**
 * Logged when compilation succeeds for a function
 */
interface CompileSuccess {
  kind: 'CompileSuccess';
  fnLoc: SourceLocation | null;
  fnName: string | null;
  memoSlots: number;
  memoBlocks: number;
  memoValues: number;
  prunedMemoBlocks: number;
  prunedMemoValues: number;
}

/**
 * Logged when an unexpected error occurs in the compilation pipeline
 */
interface PipelineError {
  kind: 'PipelineError';
  fnLoc: SourceLocation | null;
  data: Error;
}
```

### Compilation Mode

Determines which functions the compiler will process:

```typescript { .api }
/**
 * Strategy for determining which functions to compile
 */
type CompilationMode = 'infer' | 'syntax' | 'annotation' | 'all';
```

**Mode Descriptions:**

- `'infer'` (default): Compiles functions annotated with "use forget" or component/hook-like functions (named like a hook/component AND creates JSX or calls hooks)
- `'syntax'`: Compile only components using component syntax and hooks using hook syntax
- `'annotation'`: Compile only functions explicitly annotated with "use forget"
- `'all'`: Compile all top-level functions

**Opting Out:**

Any function can be opted out by adding `"use no forget"` at the top of the function body:

```javascript
function ComponentToSkip(props) {
  "use no forget";
  // This function will not be compiled
  return <div>{props.value}</div>;
}
```

### External Function Type

Used for feature gating and custom hooks configuration:

```typescript { .api }
interface ExternalFunction {
  /**
   * Module name to import from
   */
  source: string;

  /**
   * Name of the import specifier
   */
  importSpecifierName: string;
}
```

**Example (Feature Gating):**

```javascript
{
  gating: {
    source: 'ReactForgetFeatureFlag',
    importSpecifierName: 'isForgetEnabled'
  }
}
```

This produces:

```javascript
import {isForgetEnabled} from 'ReactForgetFeatureFlag';

function Component_compiled() { /* ... */ }
function Component_original() { /* ... */ }

const Component = isForgetEnabled() ? Component_compiled : Component_original;
```

### Environment Configuration

Partial environment configuration for the React Compiler. All fields are optional:

```typescript { .api }
/**
 * Partial environment configuration options from babel-plugin-react-compiler
 * All fields are optional and have defaults
 */
interface PartialEnvironmentConfig {
  /**
   * Enable experimental features (do not use in production)
   */
  enableUnstableExperimentalFeatures?: boolean;

  /**
   * Enable preservation of existing memoization guarantees
   */
  enablePreserveExistingMemoizationGuarantees?: boolean;

  /**
   * Validate preservation of existing memoization
   */
  validatePreserveExistingMemoizationGuarantees?: boolean;

  /**
   * Keep existing useMemo/useCallback calls
   */
  enablePreserveExistingManualUseMemo?: boolean;

  /**
   * Validate hooks usage rules
   */
  validateHooksUsage?: boolean;

  /**
   * Check ref.current access during render
   */
  validateRefAccessDuringRender?: boolean;

  /**
   * Prevent setState calls in render
   */
  validateNoSetStateInRender?: boolean;

  /**
   * Ensure effect dependencies are memoized
   */
  validateMemoizedEffectDependencies?: boolean;

  /**
   * Check for capitalized function calls (potential component calls)
   */
  validateNoCapitalizedCalls?: boolean;

  /**
   * Enable cache resetting when source files change (for HMR)
   */
  enableResetCacheOnSourceFileChanges?: boolean;

  /**
   * Custom hooks configuration
   */
  customHooks?: Map<string, Hook>;

  /**
   * Custom macro function names
   */
  customMacros?: Array<string>;

  // Additional environment options available in babel-plugin-react-compiler
}

/**
 * Hook configuration for custom hooks
 */
interface Hook {
  /**
   * Effect kind of the hook
   */
  effectKind: 'Read' | 'Write' | 'Freeze' | 'None';

  /**
   * Value kind produced by the hook
   */
  valueKind: 'Mutable' | 'Immutable' | 'Context' | 'Primitive' | 'Frozen';

  /**
   * Whether the hook return value should not be aliased
   */
  noAlias: boolean;

  /**
   * Whether the hook transitively returns mixed data
   */
  transitiveMixedData: boolean;
}
```

## Parser Compatibility

The plugin automatically selects the appropriate parser based on file extension:

- **TypeScript files** (`.ts`, `.tsx`): Uses `@babel/parser` with TypeScript and JSX plugins
- **JavaScript files** (`.js`, `.jsx`): Uses `hermes-parser` with experimental component syntax support

The plugin is tested with:
- `hermes-eslint` parser for JavaScript and experimental React component syntax
- Standard ESLint parsers with `@babel/parser` for TypeScript

## Flow Suppression Support

The plugin respects Flow suppression comments to avoid duplicate error reporting when Flow has already caught an issue:

```javascript
function useHookWithHook() {
  if (cond) {
    // $FlowFixMe[react-rule-hook]
    useConditionalHook(); // This error will not be reported by eslint-plugin-react-compiler
  }
}
```

Suppression comment format: `// $FlowFixMe[react-rule-hook]`

The comment must appear on the line immediately before the problematic code.

## Fix Suggestions

The plugin provides fix suggestions for certain types of violations. These suggestions can be applied automatically in supported editors or via ESLint's `--fix` option.

Suggestion types include:
- **InsertBefore**: Insert text before a specific location
- **InsertAfter**: Insert text after a specific location
- **Replace**: Replace text at a specific location
- **Remove**: Remove text at a specific location

## Compatibility

- **Node.js versions**: ^14.17.0 || ^16.0.0 || >= 18.0.0
- **ESLint versions**: >=7
- **Peer dependencies**: eslint >=7

The plugin is compatible with both older and newer ESLint APIs.

## Dependencies

The plugin relies on the following runtime dependencies:

- `@babel/core`: For AST transformation
- `@babel/parser`: For TypeScript parsing
- `@babel/plugin-proposal-private-methods`: Babel plugin for private method support
- `hermes-parser`: For JavaScript/JSX parsing
- `babel-plugin-react-compiler`: The React Compiler Babel plugin (provides core compilation logic)
- `zod`: Schema validation
- `zod-validation-error`: Validation error formatting

## Types

### Rule Module Structure

```typescript { .api }
/**
 * Standard ESLint rule module structure from the 'eslint' package
 */
interface Rule.RuleModule {
  meta: {
    type: "problem" | "suggestion" | "layout";
    docs: {
      description: string;
      recommended: boolean;
    };
    fixable?: "code" | "whitespace";
    hasSuggestions?: boolean;
    schema: any[];
  };
  create(context: Rule.RuleContext): {};
}

/**
 * ESLint rule context provided to the rule's create function
 */
interface Rule.RuleContext {
  /**
   * Source code being linted (newer ESLint API)
   */
  sourceCode?: {
    text: string;
  };

  /**
   * Get source code (older ESLint API)
   */
  getSourceCode(): {
    text: string;
    getAllComments(): CommentNode[];
  };

  /**
   * Filename being linted (newer ESLint API)
   */
  filename?: string;

  /**
   * Get filename (older ESLint API)
   */
  getFilename(): string;

  /**
   * Rule options provided in ESLint configuration
   */
  options: any[];

  /**
   * Report a linting error or warning
   */
  report(descriptor: {
    message: string;
    loc?: SourceLocation;
    suggest?: SuggestionReportDescriptor[];
  }): void;
}

/**
 * ESLint suggestion report descriptor
 */
interface Rule.SuggestionReportDescriptor {
  desc: string;
  fix(fixer: RuleFixer): Fix | null;
}
```

### Babel Source Location

```typescript { .api }
/**
 * Source location structure from Babel AST
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

### Compiler Suggestion Operations

```typescript { .api }
/**
 * Operations available for compiler suggestions
 */
enum CompilerSuggestionOperation {
  InsertBefore,
  InsertAfter,
  Replace,
  Remove
}
```

### Compiler Error Detail

Complete structure for compiler error details:

```typescript { .api }
/**
 * Detailed error information from the React Compiler
 */
interface CompilerErrorDetailOptions {
  /**
   * Primary error reason/message
   */
  reason: string;

  /**
   * Optional detailed description
   */
  description?: string | null | undefined;

  /**
   * Error severity level
   */
  severity: ErrorSeverity;

  /**
   * Source location where the error occurred
   */
  loc: SourceLocation | null;

  /**
   * Optional fix suggestions for the error
   */
  suggestions?: Array<CompilerSuggestion> | null | undefined;
}

/**
 * Source location in the code
 */
interface SourceLocation {
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

### Compiler Suggestion

Structure for code fix suggestions:

```typescript { .api }
/**
 * A suggested code fix from the compiler
 */
interface CompilerSuggestion {
  /**
   * Human-readable description of the suggestion
   */
  description: string;

  /**
   * Type of operation to perform
   */
  op: CompilerSuggestionOperation;

  /**
   * Range in the source code [start, end]
   */
  range: [number, number];

  /**
   * Text to insert or replace (not used for Remove operation)
   */
  text: string;
}
```

## Notes

- The plugin is designed to work seamlessly with the React Compiler (formerly React Forget)
- All compiler options from `babel-plugin-react-compiler` can be passed through the rule configuration
- Errors are reported using ESLint's standard reporting mechanism
- The `__unstable_donotuse_reportAllBailouts` option is experimental and subject to change
- The plugin does not emit any code; it only reports diagnostics
- Panic threshold is set to 'none' to ensure all issues are analyzed
