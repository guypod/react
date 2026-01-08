# ESLint Plugin React Compiler

ESLint plugin that surfaces diagnostics and compilation errors from React Compiler (formerly React Forget) directly in your ESLint workflow. This plugin enables developers to catch React-specific optimizations issues and violations during development by integrating the React Compiler's analysis into ESLint.

## Package Information

- **Package Name**: eslint-plugin-react-compiler
- **Package Type**: npm
- **Language**: JavaScript/TypeScript
- **Installation**: `npm install eslint-plugin-react-compiler --save-dev`
- **Peer Dependencies**: `eslint >= 7`
- **Node Version**: `^14.17.0 || ^16.0.0 || >= 18.0.0`

## Core Imports

The plugin is loaded via ESLint configuration and does not require direct imports in your code.

## Basic Usage

Add the plugin to your ESLint configuration file (`.eslintrc`, `.eslintrc.json`, or `eslint.config.js`):

**JSON Configuration:**

```json
{
  "plugins": ["react-compiler"],
  "rules": {
    "react-compiler/react-compiler": "error"
  }
}
```

**JavaScript Configuration:**

```javascript
module.exports = {
  plugins: ["react-compiler"],
  rules: {
    "react-compiler/react-compiler": "error"
  }
};
```

The plugin will analyze React code during linting and report compilation errors found by the React Compiler.

## Capabilities

### Plugin Structure

The plugin exports a standard ESLint plugin object with rules.

```javascript { .api }
/**
 * Main plugin export
 */
const plugin = {
  rules: {
    "react-compiler": ReactCompilerRule
  }
};
```

The plugin follows the standard ESLint plugin structure and exports a single rule named `react-compiler`.

### React Compiler Rule

The main rule that integrates React Compiler analysis into ESLint.

```javascript { .api }
/**
 * ESLint rule that surfaces diagnostics from React Compiler
 * Rule name: "react-compiler/react-compiler"
 *
 * This rule follows the standard ESLint rule structure with the following metadata:
 */
const ReactCompilerRule = {
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
  create: function(context) {
    // Rule implementation - parses and analyzes React code
    // using babel-plugin-react-compiler
  }
};
```

**Usage in ESLint config:**

```json
{
  "rules": {
    "react-compiler/react-compiler": "error"
  }
}
```

### Rule Options

The rule accepts an options object to customize behavior.

```javascript { .api }
/**
 * Configuration options for the react-compiler rule
 */
interface RuleOptions {
  /**
   * Set of error severity levels to report as ESLint errors
   * Default: Set containing ErrorSeverity.InvalidReact and ErrorSeverity.InvalidJS
   */
  reportableLevels?: Set<ErrorSeverity>;

  /**
   * EXPERIMENTAL: Report all compilation bailouts on the function/hook level
   * instead of on the specific line causing the issue
   * Default: false
   * Warning: This is an unstable API and may change
   */
  __unstable_donotuse_reportAllBailouts?: boolean;

  /**
   * React Compiler plugin options (passed through to babel-plugin-react-compiler)
   * See babel-plugin-react-compiler documentation for available options
   */
  environment?: EnvironmentConfig;
  logger?: Logger;
  // Additional babel-plugin-react-compiler options...
}
```

**Basic configuration with options:**

```json
{
  "rules": {
    "react-compiler/react-compiler": [
      "error",
      {
        "reportableLevels": "custom Set of ErrorSeverity values"
      }
    ]
  }
}
```

**Example with reportableLevels:**

```javascript
// In a JavaScript config file where you can use JavaScript objects
const { ErrorSeverity } = require("babel-plugin-react-compiler/src");

module.exports = {
  rules: {
    "react-compiler/react-compiler": [
      "error",
      {
        reportableLevels: new Set([
          ErrorSeverity.InvalidReact,
          ErrorSeverity.InvalidJS,
          ErrorSeverity.Todo
        ])
      }
    ]
  }
};
```

**Example with experimental bailout reporting:**

```json
{
  "rules": {
    "react-compiler/react-compiler": [
      "error",
      {
        "__unstable_donotuse_reportAllBailouts": true
      }
    ]
  }
}
```

### Error Severity Levels

Error severity levels determine which types of React Compiler diagnostics are reported.

```javascript { .api }
/**
 * Error severity levels from babel-plugin-react-compiler
 * Used in reportableLevels configuration
 */
enum ErrorSeverity {
  /**
   * Invalid React code that violates React rules
   * Default: reported
   */
  InvalidReact,

  /**
   * Invalid JavaScript code
   * Default: reported
   */
  InvalidJS,

  /**
   * Unimplemented features or TODOs in the compiler
   * Default: not reported
   */
  Todo,

  // Additional severity levels may be available in babel-plugin-react-compiler
}
```

**Note:** These severity levels are defined in `babel-plugin-react-compiler` and are re-exported for use in the ESLint plugin configuration.

### Environment Configuration

Environment configuration for React Compiler can be passed through the rule options.

```javascript { .api }
/**
 * Environment configuration for React Compiler
 * Passed through to babel-plugin-react-compiler
 * See babel-plugin-react-compiler documentation for detailed configuration options
 */
interface EnvironmentConfig {
  // Configuration options defined in babel-plugin-react-compiler
  // These control how the React Compiler analyzes and optimizes code
}
```

**Usage:**

```javascript
module.exports = {
  rules: {
    "react-compiler/react-compiler": [
      "error",
      {
        environment: {
          // Environment configuration options
        }
      }
    ]
  }
};
```

### Custom Logger

A custom logger can be provided to receive compilation events.

```javascript { .api }
/**
 * Logger interface for receiving React Compiler events
 * Passed through to babel-plugin-react-compiler
 */
interface Logger {
  /**
   * Log a compilation event
   * @param filename - The file being compiled
   * @param event - The compilation event with details
   */
  logEvent(filename: string, event: CompilerEvent): void;
}

/**
 * Compilation event from React Compiler
 */
interface CompilerEvent {
  kind: "CompileError" | "CompileSuccess" | string;
  detail?: CompilerErrorDetail;
  fnLoc?: SourceLocation;
}

/**
 * Details about a compilation error
 */
interface CompilerErrorDetail {
  severity: ErrorSeverity;
  reason: string;
  loc?: SourceLocation;
  suggestions?: Array<CompilerSuggestion>;
}

/**
 * Source location in code
 */
interface SourceLocation {
  start: { line: number; column: number };
  end: { line: number; column: number };
}
```

**Usage:**

```javascript
const customLogger = {
  logEvent(filename, event) {
    console.log(`Compilation event in ${filename}:`, event);
  }
};

module.exports = {
  rules: {
    "react-compiler/react-compiler": [
      "error",
      {
        logger: customLogger
      }
    ]
  }
};
```

### Fix Suggestions

The rule provides automatic fix suggestions for certain errors.

```javascript { .api }
/**
 * Suggestion operations that can be applied to fix issues
 * These are automatically generated by the React Compiler and surfaced through ESLint
 */
enum CompilerSuggestionOperation {
  /** Insert text before a specific range */
  InsertBefore,
  /** Insert text after a specific range */
  InsertAfter,
  /** Replace text in a specific range */
  Replace,
  /** Remove text in a specific range */
  Remove
}

/**
 * A fix suggestion for a compilation error
 */
interface CompilerSuggestion {
  op: CompilerSuggestionOperation;
  description: string;
  range: [number, number];
  text: string;
}
```

When ESLint reports an error, it may include suggestions that can be applied automatically or manually. These suggestions are generated by the React Compiler and transformed into ESLint suggestion format.

### Flow Suppression Support

The plugin respects Flow suppression comments.

**Flow suppression pattern:**

```javascript
function useHookWithHook() {
  if (cond) {
    // $FlowFixMe[react-rule-hook]
    useConditionalHook();
  }
}
```

If a Flow suppression comment with `$FlowFixMe[react-rule-hook]` is present on the line immediately before an error, the plugin will skip reporting that error, assuming Flow has already caught it.

### Parser Support

The plugin automatically selects the appropriate parser based on file extension:

- **TypeScript files (`.ts`, `.tsx`)**: Uses `@babel/parser` with TypeScript and JSX plugins
- **JavaScript files (`.js`, `.jsx`)**: Uses `hermes-parser` with experimental component syntax support

No additional parser configuration is required in most cases.

### Experimental Component Syntax

The plugin supports React's experimental component syntax when parsing with hermes-parser:

```javascript
component HelloWorld(text: string = "Hello!", onClick: () => void) {
  return <div onClick={onClick}>{text}</div>;
}
```

This syntax is automatically enabled when using JavaScript files.

## Dependencies

The plugin has the following runtime dependencies:

- `@babel/core` - For AST transformation
- `@babel/parser` - For parsing TypeScript files
- `@babel/plugin-proposal-private-methods` - For handling private class methods
- `hermes-parser` - For parsing JavaScript files with experimental syntax
- `zod` and `zod-validation-error` - For option validation
- `babel-plugin-react-compiler` (peer dependency) - The React Compiler that performs the actual analysis

**Note:** The plugin internally uses `babel-plugin-react-compiler` to perform code analysis. The plugin acts as a bridge between the React Compiler and ESLint.

## Common Error Messages

The plugin will surface various error messages from the React Compiler:

### Invalid React Patterns

**Mutating props:**
```
Mutating component props or hook arguments is not allowed. Consider using a local variable instead
```

**ESLint suppressions:**
```
React Compiler has skipped optimizing this component because one or more React ESLint rules were disabled. React Compiler only works when your components follow all the rules of React, disabling them may result in unexpected or incorrect behavior
```

### Compilation Bailouts

When using `__unstable_donotuse_reportAllBailouts`:
```
[ReactCompilerBailout] (BuildHIR::lowerStatement) Handle var kinds in VariableDeclaration (@:3:2)
```

### Unsupported Syntax

Various messages about unsupported JavaScript or React patterns that the compiler cannot optimize.

## Integration with React Compiler

This plugin is designed to work with the React Compiler (babel-plugin-react-compiler). It:

1. Parses your React code using Babel or Hermes parsers
2. Runs the React Compiler transformation on the AST
3. Captures compilation errors and diagnostics via a custom logger
4. Reports issues as ESLint errors with fix suggestions
5. Supports Flow suppression comments to avoid duplicate reporting

The plugin configures the React Compiler with:
- `noEmit: true` - Does not emit transformed code, only performs analysis
- `panicThreshold: 'none'` - Reports all errors without panicking

## Best Practices

### Configuration Recommendations

1. **Start with default severity levels**: The default `reportableLevels` (InvalidReact and InvalidJS) are suitable for most projects.

2. **Use error level**: Configure the rule as "error" to ensure React Compiler issues are treated seriously:
   ```json
   {
     "rules": {
       "react-compiler/react-compiler": "error"
     }
   }
   ```

3. **Avoid the experimental bailout flag**: The `__unstable_donotuse_reportAllBailouts` option is intended only for codebases that are 100% reliant on the React Compiler for memoization.

### Workflow Integration

1. **Add to CI/CD**: Include ESLint with this plugin in your continuous integration pipeline to catch issues early.

2. **Pre-commit hooks**: Use tools like Husky to run ESLint with this plugin before commits.

3. **IDE integration**: Configure your IDE to run ESLint automatically for real-time feedback.

### Troubleshooting

1. **No errors reported**: Ensure the React Compiler is properly installed as a peer dependency.

2. **Parser errors**: The plugin automatically selects parsers, but if you have custom parser configuration in ESLint, it may conflict.

3. **Performance concerns**: For large codebases, consider running the rule only on changed files during development.

## Example Configurations

### Minimal Configuration

```json
{
  "plugins": ["react-compiler"],
  "rules": {
    "react-compiler/react-compiler": "error"
  }
}
```

### Configuration with Custom Severity Levels

```javascript
const { ErrorSeverity } = require("babel-plugin-react-compiler/src");

module.exports = {
  plugins: ["react-compiler"],
  rules: {
    "react-compiler/react-compiler": [
      "error",
      {
        reportableLevels: new Set([
          ErrorSeverity.InvalidReact,
          ErrorSeverity.InvalidJS,
          ErrorSeverity.Todo
        ])
      }
    ]
  }
};
```

### Configuration with Custom Logger

```javascript
const fs = require("fs");
const path = require("path");

const customLogger = {
  logEvent(filename, event) {
    const logFile = path.join(__dirname, "react-compiler.log");
    fs.appendFileSync(
      logFile,
      `${new Date().toISOString()} - ${filename}: ${JSON.stringify(event)}\n`
    );
  }
};

module.exports = {
  plugins: ["react-compiler"],
  rules: {
    "react-compiler/react-compiler": [
      "error",
      {
        logger: customLogger
      }
    ]
  }
};
```

### Configuration with Environment Options

```javascript
module.exports = {
  plugins: ["react-compiler"],
  rules: {
    "react-compiler/react-compiler": [
      "error",
      {
        environment: {
          // React Compiler environment configuration
          // Refer to babel-plugin-react-compiler documentation for available options
        }
      }
    ]
  }
};
```

## Limitations

1. **Experimental status**: This plugin is in experimental stage (version 0.0.0-experimental) and APIs may change.

2. **React Compiler dependency**: Requires babel-plugin-react-compiler to be installed and properly configured.

3. **Parser limitations**: Automatic parser selection works for most cases, but custom ESLint parser configurations may interfere.

4. **Performance**: Running React Compiler analysis on every lint can be slow for large files or projects.

5. **Error reporting**: The plugin can only report errors that the React Compiler detects; it does not perform independent React analysis.
