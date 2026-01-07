# Babel Plugin

The default export provides the main Babel plugin function for integrating React Compiler into Babel-based build pipelines.

## Capabilities

### BabelPluginReactCompiler (Default Export)

The main Babel plugin function that transforms React components and hooks to optimize them with automatic memoization.

```typescript { .api }
/**
 * Creates a Babel plugin that compiles React components and hooks
 * @param babel - The Babel instance (passed automatically by Babel)
 * @returns Babel plugin object with visitor pattern for AST traversal
 */
declare function BabelPluginReactCompiler(
  babel: typeof BabelCore
): BabelCore.PluginObj;

export default BabelPluginReactCompiler;
```

**Usage Example:**

```javascript
// babel.config.js
module.exports = {
  plugins: [
    [
      "babel-plugin-react-compiler",
      {
        compilationMode: "infer",
        environment: {
          enablePreserveExistingMemoizationGuarantees: false,
        },
      },
    ],
  ],
};
```

**With Options:**

```javascript
// babel.config.js
const logger = {
  logEvent(filename, event) {
    if (event.kind === "CompileSuccess") {
      console.log(
        `Compiled ${event.fnName}: ${event.memoBlocks} memo blocks, ${event.memoValues} memoized values`
      );
    } else if (event.kind === "CompileError") {
      console.error(`Error compiling ${filename}:`, event.detail.reason);
    }
  },
};

module.exports = {
  plugins: [
    [
      "babel-plugin-react-compiler",
      {
        // Compilation strategy
        compilationMode: "infer", // 'infer' | 'syntax' | 'annotation' | 'all'

        // Error handling
        panicThreshold: "none", // 'all_errors' | 'critical_errors' | 'none'

        // Skip codegen but run analysis
        noEmit: false,

        // Custom logger for compilation events
        logger: logger,

        // Environment configuration
        environment: {
          // Preserve existing useMemo/useCallback semantics
          enablePreserveExistingMemoizationGuarantees: false,

          // Validate hooks usage
          validateHooksUsage: true,

          // Enable HMR cache reset
          enableResetCacheOnSourceFileChanges: false,

          // Custom hooks configuration
          customHooks: new Map([
            [
              "useCustomHook",
              {
                effectKind: "Read",
                valueKind: "Mutable",
                noAlias: false,
                transitiveMixedData: false,
              },
            ],
          ]),
        },

        // Feature gating
        gating: {
          source: "react-compiler-runtime",
          importSpecifierName: "isForgetEnabled",
        },

        // Custom runtime module
        runtimeModule: "react-compiler-runtime",

        // ESLint suppression checking
        eslintSuppressionRules: [
          "react-hooks/rules-of-hooks",
          "react-hooks/exhaustive-deps",
        ],

        // File filtering
        sources: (filename) => !filename.includes("node_modules"),

        // React Native Reanimated support
        enableReanimatedCheck: true,
      },
    ],
  ],
};
```

**With Feature Gating:**

When gating is configured, the compiler emits both compiled and uncompiled versions:

```javascript
{
  plugins: [
    [
      "babel-plugin-react-compiler",
      {
        gating: {
          source: "react-compiler-runtime",
          importSpecifierName: "isForgetEnabled",
        },
      },
    ],
  ];
}
```

This produces:

```javascript
import { isForgetEnabled } from "react-compiler-runtime";

function MyComponent_forget(props) {
  // Compiled version with memoization
}

function MyComponent_uncompiled(props) {
  // Original uncompiled version
}

const MyComponent = isForgetEnabled()
  ? MyComponent_forget
  : MyComponent_uncompiled;
```

**Opting Out of Compilation:**

Components can opt out using the `"use no forget"` directive:

```javascript
function MyComponent(props) {
  "use no forget";
  // This component will not be compiled
  return <div>{props.children}</div>;
}
```

**Opting In with Annotation Mode:**

```javascript
// babel.config.js with compilationMode: 'annotation'
{
  plugins: [["babel-plugin-react-compiler", { compilationMode: "annotation" }]];
}
```

```javascript
function MyComponent(props) {
  "use forget";
  // Only this function will be compiled in annotation mode
  return <div>{props.children}</div>;
}
```
