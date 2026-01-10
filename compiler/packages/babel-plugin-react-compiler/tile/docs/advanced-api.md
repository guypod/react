# Advanced API

Advanced functions for custom Babel plugin development, AST manipulation, and suppression handling. These functions are typically used when building custom integrations or extending the compiler's behavior.

## Capabilities

### Feature Gating

Insert feature-gated function declarations that conditionally use compiled or original code based on runtime flags.

```typescript { .api }
/**
 * Insert gated function declaration
 * Wraps compiled function with feature flag check that falls back to original
 * @param fnPath - Babel NodePath to function
 * @param compiled - Compiled function AST node
 * @param gating - External gating function configuration
 */
function insertGatedFunctionDeclaration(
  fnPath: NodePath<t.FunctionDeclaration | t.ArrowFunctionExpression | t.FunctionExpression>,
  compiled: t.FunctionDeclaration | t.ArrowFunctionExpression | t.FunctionExpression,
  gating: ExternalFunction
): void;
```

**Usage Example:**

```typescript
import { insertGatedFunctionDeclaration } from "babel-plugin-react-compiler";
import traverse from "@babel/traverse";

traverse(ast, {
  Function(path) {
    const compiled = compileFunction(path);

    // Gate compiled version behind feature flag
    insertGatedFunctionDeclaration(
      path,
      compiled,
      {
        source: "./featureFlags",
        importSpecifierName: "shouldUseCompiledVersion",
      }
    );
  },
});

// Generated code:
// const MyComponent = shouldUseCompiledVersion()
//   ? (props) => { /* compiled version */ }
//   : (props) => { /* original version */ };
```

### Import Management

Manage imports in the compiled program, including adding external function imports and memo cache imports.

```typescript { .api }
/**
 * Add imports to program
 * Adds import declarations for external functions at the top of the program
 * @param path - Babel NodePath to program
 * @param importList - Array of external functions to import
 */
function addImportsToProgram(
  path: NodePath<t.Program>,
  importList: Array<ExternalFunction>
): void;

/**
 * Update memo cache function import
 * Adds or updates import for React's useMemoCache (internal memo cache API)
 * @param program - Babel NodePath to program
 * @param moduleName - Module to import from (e.g., 'react')
 * @param useMemoCacheIdentifier - Local identifier name for memo cache function
 */
function updateMemoCacheFunctionImport(
  program: NodePath<t.Program>,
  moduleName: string,
  useMemoCacheIdentifier: string
): void;
```

**Usage Example:**

```typescript
import {
  addImportsToProgram,
  updateMemoCacheFunctionImport,
} from "babel-plugin-react-compiler";

traverse(ast, {
  Program(path) {
    // Add custom instrumentation imports
    addImportsToProgram(path, [
      {
        source: "./instrumentation",
        importSpecifierName: "trackRender",
      },
      {
        source: "./debugging",
        importSpecifierName: "logMemoization",
      },
    ]);

    // Add memo cache import
    updateMemoCacheFunctionImport(path, "react", "c");
  },
});

// Generated code:
// import { c } from "react";
// import { trackRender } from "./instrumentation";
// import { logMemoization } from "./debugging";
```

### Suppression Detection

Detect and handle ESLint and Flow error suppressions that may conflict with React Compiler optimizations.

```typescript { .api }
/**
 * Find program suppressions
 * Scans program comments for ESLint disable/enable and Flow suppressions
 * @param programComments - Array of comment nodes from program
 * @param ruleNames - ESLint rule names to check for suppressions
 * @param flowSuppressions - Whether to check for Flow suppressions
 * @returns Array of suppression ranges found
 */
function findProgramSuppressions(
  programComments: Array<t.Comment>,
  ruleNames: Array<string>,
  flowSuppressions: boolean
): Array<SuppressionRange>;

/**
 * Filter suppressions that affect a function
 * Returns suppressions that are either within the function or wrap it
 * @param suppressionRanges - Array of suppression ranges to filter
 * @param fn - Babel NodePath to function
 * @returns Filtered array of suppressions affecting the function
 */
function filterSuppressionsThatAffectFunction(
  suppressionRanges: Array<SuppressionRange>,
  fn: NodePath<t.Function>
): Array<SuppressionRange>;

/**
 * Convert suppressions to compiler error
 * Creates a CompilerError with details for each suppression
 * @param suppressionRanges - Array of suppression ranges
 * @returns CompilerError with suppression details, or null if no suppressions
 */
function suppressionsToCompilerError(
  suppressionRanges: Array<SuppressionRange>
): CompilerError | null;
```

**Usage Example:**

```typescript
import {
  findProgramSuppressions,
  filterSuppressionsThatAffectFunction,
  suppressionsToCompilerError,
} from "babel-plugin-react-compiler";
import * as t from "@babel/types";

// Find all suppressions in program
const allSuppressions = findProgramSuppressions(
  programComments,
  ["react-hooks/rules-of-hooks", "react-hooks/exhaustive-deps"],
  true // Check Flow suppressions
);

traverse(ast, {
  Function(path) {
    // Filter suppressions affecting this function
    const functionSuppressions = filterSuppressionsThatAffectFunction(
      allSuppressions,
      path
    );

    if (functionSuppressions.length > 0) {
      // Convert to error
      const error = suppressionsToCompilerError(functionSuppressions);

      if (error) {
        console.warn(`Cannot compile ${path.node.id?.name || "anonymous"}:`);
        console.warn(error.toString());

        // Skip compilation for this function
        return;
      }
    }
  },
});
```

## Types

### SuppressionRange

Represents a range of code with ESLint or Flow error suppressions.

```typescript { .api }
/**
 * Suppression range with disable/enable comments
 */
interface SuppressionRange {
  /**
   * Comment that disables the rule
   * For single-line suppressions, both disable and enable point to same comment
   */
  disableComment: t.Comment;

  /**
   * Comment that re-enables the rule
   * Null if suppression extends to end of file
   */
  enableComment: t.Comment | null;

  /**
   * Source of the suppression
   * - 'Eslint': ESLint disable/enable comments
   * - 'Flow': Flow error suppression comments
   */
  source: "Eslint" | "Flow";
}
```

**Example Suppressions:**

```typescript
// ESLint single-line suppression
// eslint-disable-next-line react-hooks/rules-of-hooks
useEffect(() => {}, []); // SuppressionRange: disable and enable both point to this comment

// ESLint block suppression
/* eslint-disable react-hooks/exhaustive-deps */
useEffect(() => {
  doSomething(prop);
}, []); // Missing dependency
/* eslint-enable react-hooks/exhaustive-deps */

// Flow suppression
// $FlowFixMe[react-rule-hook]
useHook();
```

## Integration Patterns

### Custom Babel Plugin with Gating

```typescript
import {
  compileProgram,
  parsePluginOptions,
  addImportsToProgram,
} from "babel-plugin-react-compiler";
import type { PluginObj } from "@babel/core";

export default function myGatedCompilerPlugin(): PluginObj {
  return {
    name: "my-gated-compiler",
    visitor: {
      Program(path, state) {
        const options = parsePluginOptions(state.opts);

        // Add gating function import
        if (options.gating) {
          addImportsToProgram(path, [options.gating]);
        }

        // Compile program
        const compilerPass = {
          opts: options,
          file: state.file,
          filename: state.filename || "unknown",
        };

        compileProgram(path, compilerPass);
      },
    },
  };
}
```

### Suppression Validation

```typescript
import {
  findProgramSuppressions,
  filterSuppressionsThatAffectFunction,
  suppressionsToCompilerError,
} from "babel-plugin-react-compiler";

function validateNoSuppressions(ast, comments) {
  const suppressions = findProgramSuppressions(
    comments,
    [
      "react-hooks/rules-of-hooks",
      "react-hooks/exhaustive-deps",
    ],
    true
  );

  if (suppressions.length > 0) {
    const error = suppressionsToCompilerError(suppressions);
    throw new Error(
      `Cannot compile: Found ${suppressions.length} React rule suppressions\n${error?.toString()}`
    );
  }
}
```

### Progressive Enhancement

```typescript
import {
  compile,
  insertGatedFunctionDeclaration,
  CompilerError,
} from "babel-plugin-react-compiler";

traverse(ast, {
  Function(path) {
    try {
      const compiled = compile(
        path,
        config,
        "Component",
        "c",
        null,
        null,
        null
      );

      // Gate behind feature flag for gradual rollout
      insertGatedFunctionDeclaration(
        path,
        toASTFunction(compiled),
        {
          source: "@company/feature-flags",
          importSpecifierName: "useCompiledReact",
        }
      );
    } catch (error) {
      if (error instanceof CompilerError) {
        // Log error but keep original code
        console.warn(`Skipping ${path.node.id?.name}:`, error.toString());
      } else {
        throw error;
      }
    }
  },
});
```

## Best Practices

### Gating Strategy

1. **Start with Opt-in**: Use gating to enable compiled code only for specific components
2. **Monitor Performance**: Track render performance and memoization effectiveness
3. **Gradual Rollout**: Increase percentage of gated users over time
4. **Fallback Safety**: Always maintain original code as fallback

### Suppression Handling

1. **Fail Early**: Check for suppressions before attempting compilation
2. **Clear Messages**: Provide actionable error messages explaining why suppressions block compilation
3. **Suggest Fixes**: Include suggestions to remove suppressions and fix underlying issues
4. **Allow Override**: Provide escape hatch for intentional suppressions (use with caution)

### Import Management

1. **Deduplicate**: Use `addImportsToProgram` to avoid duplicate imports
2. **Sort Imports**: Keep imports organized and sorted by source
3. **Check Conflicts**: Validate that imported identifiers don't conflict with existing bindings
4. **Memo Cache**: Always use `updateMemoCacheFunctionImport` when compiled code uses memo cache
