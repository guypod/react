# Plugin Configuration

Configuration options for customizing the React Compiler's behavior, including plugin options, environment configuration, and feature flags.

## Capabilities

### Parse Plugin Options

Parse and validate plugin options from an object.

```typescript { .api }
/**
 * Parse plugin options from object
 * @param obj - Options object to parse
 * @returns Validated PluginOptions
 * @throws CompilerError if options are invalid
 */
function parsePluginOptions(obj: unknown): PluginOptions;
```

**Usage Example:**

```typescript
import { parsePluginOptions } from "babel-plugin-react-compiler";

const options = parsePluginOptions({
  compilationMode: "infer",
  panicThreshold: "all_errors",
  environment: {
    validateHooksUsage: true,
    enableInstructionReordering: false,
  },
});
```

### Parse Configuration Pragma

Parse configuration from pragma comments in source code.

```typescript { .api }
/**
 * Parse configuration from pragma comments
 * @param pragma - Pragma string (JSON format)
 * @returns Parsed environment configuration
 * @example parseConfigPragma('{"validateHooksUsage": true}')
 */
function parseConfigPragma(pragma: string): EnvironmentConfig;
```

**Usage Example:**

```typescript
import { parseConfigPragma } from "babel-plugin-react-compiler";

// Parse from a pragma comment like:
// @react-forget {"validateHooksUsage": true, "enablePreserveExistingMemoizationGuarantees": false}
const config = parseConfigPragma('{"validateHooksUsage": true}');
```

### Validate Environment Configuration

Validate environment configuration against schema.

```typescript { .api }
/**
 * Validate environment configuration
 * @param config - Configuration object to validate
 * @returns Result with validated config or Zod validation error
 */
function validateEnvironmentConfig(
  config: unknown
): Result<EnvironmentConfig, ZodError>;
```

## Types

### PluginOptions

Main configuration object for the Babel plugin.

```typescript { .api }
interface PluginOptions {
  /**
   * Environment configuration for compiler behavior
   * Null uses default configuration
   */
  environment: PartialEnvironmentConfig | null;

  /**
   * Logger for compilation events
   * Null disables logging
   */
  logger: Logger | null;

  /**
   * Feature flag gating configuration
   * External function called to check if compilation should run
   */
  gating: ExternalFunction | null;

  /**
   * Error threshold configuration
   * - 'all_errors': Panic on any error
   * - 'critical_errors': Only panic on critical errors
   * - 'none': Never panic, always emit code
   */
  panicThreshold: "all_errors" | "critical_errors" | "none";

  /**
   * Skip code generation (analysis only)
   * When true, runs compilation but doesn't emit optimized code
   */
  noEmit: boolean;

  /**
   * Compilation strategy
   * - 'infer': Auto-detect React components/hooks
   * - 'syntax': Only compile components with specific syntax markers
   * - 'annotation': Only compile annotated functions
   * - 'all': Compile all functions
   */
  compilationMode: "infer" | "syntax" | "annotation" | "all";

  /**
   * Custom runtime module path
   * Override default react runtime import path
   */
  runtimeModule?: string | null;

  /**
   * ESLint suppression rules to check
   * Array of rule names to validate aren't suppressed
   */
  eslintSuppressionRules?: Array<string> | null;

  /**
   * Flow suppression checking
   * Check for Flow error suppressions
   */
  flowSuppressions: boolean;

  /**
   * Ignore "use no forget" directives
   * For testing only - ignores opt-out directives
   */
  ignoreUseNoForget: boolean;

  /**
   * Source file filtering
   * Can be array of glob patterns or filter function
   */
  sources?: Array<string> | ((filename: string) => boolean) | null;

  /**
   * React Native Reanimated support
   * Enable special handling for Reanimated hooks
   */
  enableReanimatedCheck: boolean;
}
```

**Usage Example:**

```typescript
const pluginOptions: PluginOptions = {
  environment: {
    validateHooksUsage: true,
    enablePreserveExistingMemoizationGuarantees: true,
  },
  logger: {
    logEvent: (filename, event) => {
      console.log(`[${event.kind}] ${filename}:`, event);
    },
  },
  gating: null,
  panicThreshold: "all_errors",
  noEmit: false,
  compilationMode: "infer",
  runtimeModule: null,
  eslintSuppressionRules: ["react-hooks/exhaustive-deps"],
  flowSuppressions: false,
  ignoreUseNoForget: false,
  sources: ["src/**/*.tsx", "src/**/*.ts"],
  enableReanimatedCheck: false,
};
```

### EnvironmentConfig

Extensive configuration for fine-grained compiler behavior control (70+ options).

```typescript { .api }
interface EnvironmentConfig {
  // === Custom Hooks & Macros ===

  /**
   * Custom hook definitions
   * Map of hook names to Hook configurations
   */
  customHooks: Map<string, Hook>;

  /**
   * Macro function names
   * Array of function names treated as compile-time macros
   */
  customMacros: Array<string> | null;

  // === Memoization Behavior ===

  /**
   * Preserve existing memoization guarantees
   * Ensures compiled code maintains same memoization as original
   */
  enablePreserveExistingMemoizationGuarantees: boolean;

  /**
   * Validate preservation of memoization guarantees
   * Throws error if existing memoization can't be preserved
   */
  validatePreserveExistingMemoizationGuarantees: boolean;

  /**
   * Preserve manual useMemo/useCallback
   * Keeps explicit memoization calls instead of replacing them
   */
  enablePreserveExistingManualUseMemo: boolean;

  /**
   * Disable memoization for debugging
   * Compiles but skips memoization code generation
   */
  disableMemoizationForDebugging: boolean;

  // === Type System ===

  /**
   * Use type annotations for optimization
   * Leverages TypeScript/Flow types for better analysis
   */
  enableUseTypeAnnotations: boolean;

  /**
   * Enable reactive scopes in HIR
   * Include reactive scope information in HIR stage
   */
  enableReactiveScopesInHIR: boolean;

  // === Validation Options ===

  /**
   * Validate hooks follow Rules of React
   * Checks hooks are called unconditionally at top level
   */
  validateHooksUsage: boolean;

  /**
   * Validate refs not accessed during render
   * Ensures ref.current not read during render phase
   */
  validateRefAccessDuringRender: boolean;

  /**
   * Validate no setState in render
   * Prevents setState calls during render
   */
  validateNoSetStateInRender: boolean;

  /**
   * Validate memoized effect dependencies
   * Checks effect dependencies are properly memoized
   */
  validateMemoizedEffectDependencies: boolean;

  /**
   * Validate no capitalized function calls
   * Array of patterns for functions that shouldn't be called like components
   */
  validateNoCapitalizedCalls: Array<string> | null;

  // === Optimization Features ===

  /**
   * Enable instruction reordering
   * Reorder instructions for better optimization
   */
  enableInstructionReordering: boolean;

  /**
   * Enable function outlining
   * Extract functions for better code splitting
   */
  enableFunctionOutlining: boolean;

  /**
   * Assume hooks follow React rules
   * Skip some validation for performance
   */
  enableAssumeHooksFollowRulesOfReact: boolean;

  /**
   * Transitively freeze function expressions
   * Freeze nested function expressions for optimization
   */
  enableTransitivelyFreezeFunctionExpressions: boolean;

  // === Debugging & Instrumentation ===

  /**
   * Emit freeze calls for debugging
   * Add runtime freeze() calls via external function
   */
  enableEmitFreeze: ExternalFunction | null;

  /**
   * Emit hook guards for debugging
   * Add runtime checks for hook call validity
   */
  enableEmitHookGuards: ExternalFunction | null;

  /**
   * Emit instrumentation for Forget
   * Add instrumentation code according to schema
   */
  enableEmitInstrumentForget: InstrumentationSchema | null;

  /**
   * Change detection for debugging
   * Add change detection via external function
   */
  enableChangeDetectionForDebugging: ExternalFunction | null;

  /**
   * Add memoization comments to output
   * Include comments explaining memoization decisions
   */
  enableMemoizationComments: boolean;

  /**
   * Enable change variable codegen
   * Generate $[changed] variables for debugging
   */
  enableChangeVariableCodegen: boolean;

  // === Advanced Settings ===

  /**
   * Custom hook name pattern
   * Regex pattern for identifying hooks (default: /^use[A-Z]/)
   */
  hookPattern: string | null;

  /**
   * Treat ref-like identifiers as refs
   * Variables matching *Ref pattern treated as refs
   */
  enableTreatRefLikeIdentifiersAsRefs: boolean;

  /**
   * Custom type definition for Reanimated
   * Enable React Native Reanimated type handling
   */
  enableCustomTypeDefinitionForReanimated: boolean;

  /**
   * Reset cache on source file changes
   * Enable HMR support by invalidating cache
   */
  enableResetCacheOnSourceFileChanges: boolean;

  /**
   * Enable optional chaining invocation
   * Support foo?.() syntax
   */
  enableOptionalChainInvocation: boolean;

  /**
   * Infer mutable ranges
   * Automatically determine where values can mutate
   */
  enableInferMutableRanges: boolean;

  /**
   * Enable do expressions
   * Support do { } expression syntax
   */
  enableDoExpressions: boolean;

  /**
   * Enable forest parsing mode
   * Experimental multi-tree parsing
   */
  enableForest: boolean;

  /**
   * Ignore function components returning null
   * Skip optimization for null-returning components
   */
  enableIgnoreFunctionComponentsReturningNull: boolean;

  /**
   * Skip non-Forget memo imports
   * Don't process memo imports from other sources
   */
  enableSkipNonForgetMemoImports: boolean;

  /**
   * Emit compiled sources
   * Include compilation metadata in output
   */
  enableEmitCompiledSources: boolean;

  /**
   * Share memoization cache across scopes
   * Use shared memo cache for better performance
   */
  enableSharedMemoCache: boolean;

  /**
   * Enable JSX memo validation
   * Validate JSX elements for memoization
   */
  enableJsxMemoValidation: boolean;

  /**
   * React Server Components mode
   * Enable RSC-specific optimizations
   */
  enableReactServerComponents: boolean;

  /**
   * Experimental React Forget runtime
   * Use experimental runtime features
   */
  enableExperimentalForgetRuntime: boolean;

  /**
   * Refine dependencies
   * More precise dependency tracking
   */
  enableRefineDeps: boolean;

  /**
   * Prune unused scopes
   * Remove scopes with no observable effects
   */
  enablePruneUnusedScopes: boolean;

  /**
   * Prune unused lambdas
   * Remove unused function expressions
   */
  enablePruneUnusedLambdas: boolean;

  /**
   * Drop manual memoization
   * Remove manual useMemo/useCallback when redundant
   */
  enableDropManualMemoization: boolean;

  /**
   * Merge consecutive reactive scopes
   * Combine adjacent scopes for efficiency
   */
  enableMergeConsecutiveScopes: boolean;

  /**
   * Fire forgotten conditional dependency violation
   * Error on forgotten conditional dependencies
   */
  enableFireForgettenConditionalDependencyViolation: boolean;

  /**
   * Validate no JSX in TRY
   * Prevent JSX in try blocks (experimental)
   */
  validateNoJSXInTRY: boolean;

  /**
   * Validate scope variables do not shadow
   * Prevent variable shadowing in scopes
   */
  validateScopeVariablesDoNotShadow: boolean;

  /**
   * Validate module naming convention
   * Enforce naming patterns for modules
   */
  validateModuleNamingConvention: boolean;

  /**
   * Validate no reference before declaration
   * Prevent TDZ issues
   */
  validateNoReferenceBeforeDeclaration: boolean;
}
```

**Common Configuration Examples:**

```typescript
import type { EnvironmentConfig } from "babel-plugin-react-compiler";

// Production configuration
const productionConfig: Partial<EnvironmentConfig> = {
  validateHooksUsage: true,
  enablePreserveExistingMemoizationGuarantees: true,
  enableInstructionReordering: true,
  panicThreshold: "critical_errors",
};

// Development configuration
const devConfig: Partial<EnvironmentConfig> = {
  validateHooksUsage: true,
  validateRefAccessDuringRender: true,
  validateNoSetStateInRender: true,
  enableMemoizationComments: true,
  enableChangeVariableCodegen: true,
};

// Debugging configuration
const debugConfig: Partial<EnvironmentConfig> = {
  disableMemoizationForDebugging: true,
  enableMemoizationComments: true,
  enableChangeVariableCodegen: true,
};
```

### Hook Configuration

Configuration for custom hooks.

```typescript { .api }
interface Hook {
  /**
   * Effect on hook arguments
   * How the hook affects its arguments
   */
  effectKind: Effect;

  /**
   * Return value kind
   * What kind of value the hook returns
   */
  valueKind: ValueKind;

  /**
   * Prevent aliasing
   * Whether return value can be aliased
   */
  noAlias: boolean;

  /**
   * Returns JSON-like transitive data
   * Whether return value is JSON-serializable
   */
  transitiveMixedData: boolean;
}
```

**Usage Example:**

```typescript
import { Effect, ValueKind, type Hook } from "babel-plugin-react-compiler";

// Define a custom hook configuration
const customHook: Hook = {
  effectKind: Effect.Read,
  valueKind: ValueKind.Mutable,
  noAlias: false,
  transitiveMixedData: false,
};

// Use in environment config
const config = {
  customHooks: new Map([["useCustomData", customHook]]),
};
```

### ExternalFunction

Reference to an external function for feature gating or instrumentation.

```typescript { .api }
interface ExternalFunction {
  /**
   * Import source module
   * Module path to import from
   */
  source: string;

  /**
   * Function name to import
   * Specific export name from the module
   */
  importSpecifierName: string;
}
```

**Usage Example:**

```typescript
import type { ExternalFunction } from "babel-plugin-react-compiler";

// Feature gating function
const gatingFunction: ExternalFunction = {
  source: "./featureFlags",
  importSpecifierName: "shouldCompileComponent",
};

// Freeze function for debugging
const freezeFunction: ExternalFunction = {
  source: "react-compiler-runtime",
  importSpecifierName: "$freeze",
};

// Use in plugin options
const options = {
  gating: gatingFunction,
  environment: {
    enableEmitFreeze: freezeFunction,
  },
};
```

### InstrumentationSchema

Schema for instrumentation configuration.

```typescript { .api }
interface InstrumentationSchema {
  /**
   * Function to call for instrumentation
   */
  fn: ExternalFunction;

  /**
   * Instrumentation options
   */
  options?: Record<string, unknown>;
}
```

## Configuration Best Practices

### Start with Defaults

Begin with default configuration and enable features incrementally:

```typescript
const options = {
  compilationMode: "infer",
  panicThreshold: "all_errors",
  environment: null, // Use defaults
};
```

### Enable Validation in Development

Use strict validation during development:

```typescript
const options = {
  environment: {
    validateHooksUsage: true,
    validateRefAccessDuringRender: true,
    validateNoSetStateInRender: true,
    validateMemoizedEffectDependencies: true,
  },
};
```

### Optimize for Production

Enable optimizations for production builds:

```typescript
const options = {
  environment: {
    enableInstructionReordering: true,
    enableFunctionOutlining: true,
    enablePruneUnusedScopes: true,
    enablePruneUnusedLambdas: true,
    enableMergeConsecutiveScopes: true,
  },
};
```

### Add Debugging Instrumentation

Enable detailed debugging when troubleshooting:

```typescript
const options = {
  environment: {
    enableMemoizationComments: true,
    enableChangeVariableCodegen: true,
    enableEmitInstrumentForget: {
      fn: {
        source: "./instrumentation",
        importSpecifierName: "instrument",
      },
    },
  },
};
```
