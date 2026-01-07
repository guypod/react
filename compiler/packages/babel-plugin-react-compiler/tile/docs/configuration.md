# Configuration

Comprehensive configuration options for controlling React Compiler behavior, validation rules, optimization strategies, and error handling.

## Capabilities

### parsePluginOptions

Parses and validates plugin options from raw input, applying defaults and validating the structure.

```typescript { .api }
/**
 * Parses and validates plugin options
 * @param obj - Raw options object (typically from Babel config)
 * @returns Validated and normalized options with defaults applied
 * @throws On invalid configuration
 */
function parsePluginOptions(obj: unknown): PluginOptions;
```

**Usage Example:**

```typescript
import { parsePluginOptions } from "babel-plugin-react-compiler";

const options = parsePluginOptions({
  compilationMode: "infer",
  environment: {
    validateHooksUsage: true,
  },
});

console.log(options.panicThreshold); // 'none' (default)
console.log(options.noEmit); // false (default)
```

---

### PluginOptions

Main configuration interface for the React Compiler plugin.

```typescript { .api }
type PluginOptions = {
  /** Environment configuration for the compiler */
  environment: PartialEnvironmentConfig | null;

  /** Logger for compilation events */
  logger: Logger | null;

  /**
   * Feature gating configuration
   * Emits separate compiled and uncompiled versions gated by an import
   */
  gating: ExternalFunction | null;

  /**
   * Error handling strategy:
   * - 'all_errors': Panic on any error
   * - 'critical_errors': Panic only on critical errors
   * - 'none': Never panic (default)
   */
  panicThreshold: "all_errors" | "critical_errors" | "none";

  /**
   * Skip codegen but run analysis
   * When true, performs compilation analysis without emitting transformed code
   * @default false
   */
  noEmit: boolean;

  /**
   * Strategy for determining which functions to compile:
   * - 'infer': Compile "use forget" or component/hook-like functions (default)
   * - 'syntax': Compile only component/hook syntax declarations
   * - 'annotation': Compile only "use forget" annotated functions
   * - 'all': Compile all top-level functions
   */
  compilationMode: "infer" | "syntax" | "annotation" | "all";

  /**
   * Custom module for useMemoCache import
   * If set, imports from this module instead of 'react/compiler-runtime'
   * @example "react-compiler-runtime"
   */
  runtimeModule?: string | null | undefined;

  /**
   * ESLint rules to check for suppression
   * Code suppressing these rules will skip compilation
   * Pass empty array to disable this feature
   * @default ['react-hooks/rules-of-hooks', 'react-hooks/exhaustive-deps']
   */
  eslintSuppressionRules?: Array<string> | null | undefined;

  /**
   * Enable Flow suppression checking
   * @default false
   */
  flowSuppressions: boolean;

  /**
   * Ignore 'use no forget' annotations
   * Useful for testing but should not be used in production
   * @default false
   */
  ignoreUseNoForget: boolean;

  /**
   * File filter for compilation
   * Array of file paths or function to determine which files to compile
   * @default filename => filename.indexOf('node_modules') === -1
   */
  sources?: Array<string> | ((filename: string) => boolean) | null;

  /**
   * Enable react-native-reanimated support detection
   * @default true
   */
  enableReanimatedCheck: boolean;
};
```

**Default Values:**

```typescript
{
  compilationMode: 'infer',
  panicThreshold: 'none',
  environment: {},
  logger: null,
  gating: null,
  noEmit: false,
  runtimeModule: null,
  eslintSuppressionRules: null,
  flowSuppressions: false,
  ignoreUseNoForget: false,
  sources: filename => filename.indexOf('node_modules') === -1,
  enableReanimatedCheck: true,
}
```

---

### Logger

Interface for logging compilation events.

```typescript { .api }
type Logger = {
  logEvent: (filename: string | null, event: LoggerEvent) => void;
};

type LoggerEvent =
  | {
      kind: "CompileError";
      fnLoc: t.SourceLocation | null;
      detail: CompilerErrorDetailOptions;
    }
  | {
      kind: "CompileDiagnostic";
      fnLoc: t.SourceLocation | null;
      detail: {
        reason: string;
        description?: string | null;
        loc: SourceLocation | null;
      };
    }
  | {
      kind: "CompileSuccess";
      fnLoc: t.SourceLocation | null;
      fnName: string | null;
      memoSlots: number;
      memoBlocks: number;
      memoValues: number;
      prunedMemoBlocks: number;
      prunedMemoValues: number;
    }
  | {
      kind: "PipelineError";
      fnLoc: t.SourceLocation | null;
      data: string;
    };
```

**Usage Example:**

```typescript
const logger = {
  logEvent(filename, event) {
    switch (event.kind) {
      case "CompileSuccess":
        console.log(
          `✓ ${event.fnName || "anonymous"} (${filename}):`,
          `${event.memoBlocks} scopes, ${event.memoValues} values`
        );
        break;

      case "CompileError":
        console.error(
          `✗ Compilation error (${filename}):`,
          event.detail.reason,
          event.detail.description
        );
        break;

      case "CompileDiagnostic":
        console.warn(`⚠ ${event.detail.reason}`);
        break;

      case "PipelineError":
        console.error(`Pipeline error (${filename}):`, event.data);
        break;
    }
  },
};

// Use in configuration
{
  plugins: [["babel-plugin-react-compiler", { logger }]];
}
```

---

### ExternalFunction

Configuration for importing external functions, used for feature gating and instrumentation.

```typescript { .api }
type ExternalFunction = {
  /** Module to import from */
  source: string;
  /** Name of the import */
  importSpecifierName: string;
};
```

**Usage Example:**

```typescript
// Feature gating
const gating: ExternalFunction = {
  source: "react-compiler-runtime",
  importSpecifierName: "isForgetEnabled",
};

// Produces:
// import { isForgetEnabled } from 'react-compiler-runtime';
// const MyComponent = isForgetEnabled() ? MyComponent_forget : MyComponent_uncompiled;
```

---

### EnvironmentConfig

Comprehensive environment configuration with 50+ options for fine-grained control over compiler behavior.

```typescript { .api }
type EnvironmentConfig = {
  /** Custom hooks configuration map (hook name -> Hook config) */
  customHooks: Map<string, Hook>;

  /**
   * Macro functions that should not be renamed
   * Functions like featureflag("name") that are rewritten by other plugins
   */
  customMacros: Array<string> | null;

  /**
   * Enable HMR cache reset when source code changes
   * Supports hot module reloading by resetting memoization cache
   * @default false
   */
  enableResetCacheOnSourceFileChanges: boolean;

  /**
   * Preserve existing useMemo/useCallback memoization guarantees
   * When true, compiler respects manual memoization boundaries
   * @default false
   */
  enablePreserveExistingMemoizationGuarantees: boolean;

  /**
   * Validate that manual memoization is preserved
   * @default true
   */
  validatePreserveExistingMemoizationGuarantees: boolean;

  /**
   * Keep existing useMemo/useCallback calls instead of pruning them
   * @default false
   */
  enablePreserveExistingManualUseMemo: boolean;

  /**
   * Enable Forest mode
   * @default false
   */
  enableForest: boolean;

  /**
   * Trust user type annotations for inference
   * @default false
   */
  enableUseTypeAnnotations: boolean;

  /**
   * Enable reactive scopes in HIR
   * @default false
   */
  enableReactiveScopesInHIR: boolean;

  /**
   * Validate hooks usage follows Rules of React
   * @default true
   */
  validateHooksUsage: boolean;

  /**
   * Validate ref.current is not accessed during render
   * @default true
   */
  validateRefAccessDuringRender: boolean;

  /**
   * Validate setState is not called during render
   * @default true
   */
  validateNoSetStateInRender: boolean;

  /**
   * Validate effect dependencies are memoized
   * @default true
   */
  validateMemoizedEffectDependencies: boolean;

  /**
   * Validate no capitalized function calls (with allowlist)
   * Array of allowed capitalized function names
   * null to disable validation
   */
  validateNoCapitalizedCalls: Array<string> | null;

  /**
   * Assume hooks follow Rules of React without deep analysis
   * @default false
   */
  enableAssumeHooksFollowRulesOfReact: boolean;

  /**
   * Transitively freeze function expression captures
   * @default false
   */
  enableTransitivelyFreezeFunctionExpressions: boolean;

  /**
   * Emit freeze calls for debugging mutations
   * Specifies function to import and call on frozen values
   */
  enableEmitFreeze: ExternalFunction | null;

  /**
   * Emit hook guards
   * Specifies function to import and call for hook validation
   */
  enableEmitHookGuards: ExternalFunction | null;

  /**
   * Enable instruction reordering optimization
   * @default true
   */
  enableInstructionReordering: boolean;

  /**
   * Enable function outlining optimization
   * Extract inline functions to top level
   * @default false
   */
  enableFunctionOutlining: boolean;

  /**
   * Emit instrumentation calls for monitoring
   */
  enableEmitInstrumentForget: {
    fn: ExternalFunction;
    gating?: ExternalFunction | null;
    globalGating?: string | null;
  } | null;

  /**
   * Validate mutable ranges (internal debugging)
   * @default false
   */
  assertValidMutableRanges: boolean;

  /**
   * Emit change variables for dependencies
   * @default false
   */
  enableChangeVariableCodegen: boolean;

  /**
   * Emit comments explaining compilation decisions
   * @default false
   */
  enableMemoizationComments: boolean;

  /**
   * Throw exception during compilation (test only)
   * @default false
   */
  throwUnknownException__testonly: boolean;

  /**
   * Enable shared runtime (test only)
   * @default false
   */
  enableSharedRuntime__testonly: boolean;

  /**
   * Treat function dependencies as conditional
   * @default false
   */
  enableTreatFunctionDepsAsConditional: boolean;

  /**
   * Disable memoization for debugging
   * @default false
   */
  disableMemoizationForDebugging: boolean;

  /**
   * Enable change detection for debugging
   * Specifies function to call when values change
   */
  enableChangeDetectionForDebugging: ExternalFunction | null;

  /**
   * Enable react-native-reanimated custom type definitions
   * @default false
   */
  enableCustomTypeDefinitionForReanimated: boolean;

  /**
   * Custom hook name pattern for matching
   * Regular expression pattern
   */
  hookPattern: string | null;

  /**
   * Treat ref-like identifiers as refs
   * Identifiers matching patterns like *Ref, *ref
   */
  enableTreatRefLikeIdentifiersAsRefs: boolean | null;
};

type PartialEnvironmentConfig = Partial<EnvironmentConfig>;
```

**Usage Example:**

```typescript
const environment: PartialEnvironmentConfig = {
  // Validation
  validateHooksUsage: true,
  validateRefAccessDuringRender: true,
  validateNoSetStateInRender: true,

  // Optimization
  enableInstructionReordering: true,
  enableFunctionOutlining: false,

  // Debugging
  enableMemoizationComments: true,
  disableMemoizationForDebugging: false,

  // Custom hooks
  customHooks: new Map([
    [
      "useQuery",
      {
        effectKind: "Read",
        valueKind: "Mutable",
        noAlias: false,
        transitiveMixedData: true,
      },
    ],
  ]),

  // Preserve manual memoization
  enablePreserveExistingMemoizationGuarantees: true,
  validatePreserveExistingMemoizationGuarantees: true,
};

// Use in plugin options
{
  plugins: [["babel-plugin-react-compiler", { environment }]];
}
```

---

### Hook

Configuration for custom hooks.

```typescript { .api }
type Hook = {
  /**
   * Effect of hook arguments:
   * - 'freeze': Freezes the value
   * - 'read': Reads the value
   * - 'capture': Reads and stores
   * - 'mutate?': May mutate
   * - 'mutate': Does mutate
   * - 'store': May alias
   */
  effectKind: Effect;

  /**
   * Kind of return value:
   * - 'frozen': Definitely frozen
   * - 'maybefrozen': May or may not be frozen
   * - 'primitive': Primitive value
   * - 'mutable': Mutable value
   * - 'global': Global value
   * - 'context': Context value
   */
  valueKind: ValueKind;

  /**
   * Whether hook arguments may be aliased
   * When true, compiler avoids memoizing arguments
   * @default false
   */
  noAlias: boolean;

  /**
   * Whether return data is transitively mixed (JSON-like)
   * Enables better optimization for data-fetching hooks
   * @default false
   */
  transitiveMixedData: boolean;
};
```

**Usage Example:**

```typescript
// Data fetching hook that returns JSON-like data
const useQuery: Hook = {
  effectKind: "Read",
  valueKind: "Mutable",
  noAlias: false,
  transitiveMixedData: true, // Returns JSON data
};

// State management hook
const useStore: Hook = {
  effectKind: "Store",
  valueKind: "Mutable",
  noAlias: false,
  transitiveMixedData: false,
};

// Pure computation hook
const useMemo: Hook = {
  effectKind: "Freeze",
  valueKind: "Frozen",
  noAlias: true,
  transitiveMixedData: false,
};

const environment = {
  customHooks: new Map([
    ["useQuery", useQuery],
    ["useStore", useStore],
    ["useComputedValue", useMemo],
  ]),
};
```

---

### validateEnvironmentConfig

Validates and normalizes a partial environment configuration, applying defaults and checking for invalid combinations.

```typescript { .api }
/**
 * Validates and normalizes environment configuration
 * @param partialConfig - Partial environment configuration
 * @returns Validated full configuration with defaults
 * @throws CompilerError on invalid configuration
 */
function validateEnvironmentConfig(
  partialConfig: PartialEnvironmentConfig
): EnvironmentConfig;
```

**Usage Example:**

```typescript
import {
  validateEnvironmentConfig,
  CompilerError,
} from "babel-plugin-react-compiler";

try {
  const config = validateEnvironmentConfig({
    validateHooksUsage: true,
    enableInstructionReordering: true,
    customHooks: new Map([
      [
        "useData",
        {
          effectKind: "Read",
          valueKind: "Mutable",
          noAlias: false,
          transitiveMixedData: true,
        },
      ],
    ]),
  });

  console.log(config.validateRefAccessDuringRender); // true (default)
} catch (err) {
  if (err instanceof CompilerError) {
    console.error("Invalid configuration:", err.toString());
  }
}
```

---

### parseConfigPragma

Parses environment configuration from pragma strings, typically from special comments in source code.

```typescript { .api }
/**
 * Parses environment config from pragma string
 * @param pragma - Space-separated config flags starting with @
 * @returns Parsed environment configuration
 * @example parseConfigPragma('@enableForest @validateHooksUsage')
 */
function parseConfigPragma(pragma: string): EnvironmentConfig;
```

**Usage Example:**

```typescript
import { parseConfigPragma } from "babel-plugin-react-compiler";

// From comment pragma
const config1 = parseConfigPragma("@enableForest @validateHooksUsage");

// Multiple flags
const config2 = parseConfigPragma(
  "@enableMemoizationComments @enableInstructionReordering @validateRefAccessDuringRender"
);

console.log(config1.enableForest); // true
console.log(config1.validateHooksUsage); // true
console.log(config2.enableMemoizationComments); // true
```

**Pragma in Source Code:**

```javascript
/**
 * @enableForest @validateHooksUsage
 */
function MyComponent() {
  // Compiled with forest mode and hooks validation enabled
}
```
