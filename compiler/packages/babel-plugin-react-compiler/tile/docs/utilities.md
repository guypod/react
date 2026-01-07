# Utility Functions

Helper functions for debugging, printing intermediate representations, and configuration management.

## Capabilities

### printHIR

Pretty-prints HIR (High-level Intermediate Representation) control flow graph. Useful for debugging and understanding how the compiler analyzes code.

```typescript { .api }
/**
 * Pretty-prints HIR control flow graph
 * @param ir - The HIR to print
 * @param options - Formatting options
 * @returns String representation of the HIR
 */
function printHIR(ir: HIR, options?: { indent?: number } | null): string;
```

**Usage Examples:**

```typescript
import { run, printHIR, validateEnvironmentConfig } from "babel-plugin-react-compiler";
import * as babel from "@babel/core";
import traverse from "@babel/traverse";

const code = `
function MyComponent({ items }) {
  const filtered = items.filter(item => item.active);
  const count = filtered.length;
  return <div>Active: {count}</div>;
}
`;

const ast = babel.parse(code, { sourceType: "module", plugins: ["jsx"] });

traverse(ast, {
  FunctionDeclaration(path) {
    const config = validateEnvironmentConfig({});
    const generator = run(
      path,
      config,
      "Component",
      "useMemoCache",
      null,
      "MyComponent.js",
      code
    );

    // Print HIR stages
    for (const stage of generator) {
      if (stage.kind === "hir") {
        console.log(`\n=== ${stage.name} ===`);
        console.log(printHIR(stage.value));

        // With custom indentation
        console.log("\nWith indent=4:");
        console.log(printHIR(stage.value, { indent: 4 }));
      }
    }
  },
});
```

**Example HIR Output:**

```
bb0 (block):
  [1] %0 = LoadLocal items
  [2] %1 = LoadGlobal filter
  [3] %2 = CallExpression %1(%0, <arrow function>)
  [4] StoreLocal filtered = %2
  [5] %3 = LoadLocal filtered
  [6] %4 = PropertyLoad %3.length
  [7] StoreLocal count = %4
  [8] %5 = JSXElement <div>
  [9] Return %5
```

**Analyzing HIR:**

```typescript
import { run, printHIR } from "babel-plugin-react-compiler";

function analyzeCompilation(functionPath, config) {
  const generator = run(
    functionPath,
    config,
    "Component",
    "useMemoCache",
    null,
    "component.js",
    null
  );

  const hirStages = [];

  for (const stage of generator) {
    if (stage.kind === "hir") {
      hirStages.push({
        name: stage.name,
        hir: printHIR(stage.value),
      });
    }
  }

  // Compare HIR before and after optimizations
  console.log("Initial HIR:");
  console.log(hirStages[0].hir);

  console.log("\nOptimized HIR:");
  console.log(hirStages[hirStages.length - 1].hir);

  return hirStages;
}
```

---

### printReactiveFunction

Pretty-prints a reactive function with its scopes, dependencies, and memoization blocks. Essential for understanding how the compiler creates reactive scopes.

```typescript { .api }
/**
 * Pretty-prints a reactive function with its scopes
 * @param fn - The reactive function to print
 * @returns String representation of the reactive function
 */
function printReactiveFunction(fn: ReactiveFunction): string;
```

**Usage Examples:**

```typescript
import {
  run,
  printReactiveFunction,
  validateEnvironmentConfig,
} from "babel-plugin-react-compiler";
import * as babel from "@babel/core";
import traverse from "@babel/traverse";

const code = `
function MyComponent({ user, settings }) {
  const displayName = user.firstName + ' ' + user.lastName;
  const theme = settings.theme || 'light';

  return (
    <div className={theme}>
      <h1>{displayName}</h1>
    </div>
  );
}
`;

const ast = babel.parse(code, { sourceType: "module", plugins: ["jsx"] });

traverse(ast, {
  FunctionDeclaration(path) {
    const config = validateEnvironmentConfig({
      enableMemoizationComments: true,
    });

    const generator = run(
      path,
      config,
      "Component",
      "useMemoCache",
      null,
      "MyComponent.js",
      code
    );

    // Print reactive stages
    for (const stage of generator) {
      if (stage.kind === "reactive") {
        console.log(`\n=== ${stage.name} ===`);
        console.log(printReactiveFunction(stage.value));
      }
    }
  },
});
```

**Example Reactive Function Output:**

```
function MyComponent(user, settings) {
  scope @0 [1:3] {
    deps: [user.firstName, user.lastName]
    declarations: [displayName]

    displayName = user.firstName + ' ' + user.lastName;
  }

  scope @1 [4:4] {
    deps: [settings.theme]
    declarations: [theme]

    theme = settings.theme || 'light';
  }

  scope @2 [6:10] {
    deps: [theme, displayName]
    declarations: [jsx]

    jsx = <div className={theme}>
      <h1>{displayName}</h1>
    </div>;
  }

  return jsx;
}
```

**Analyzing Reactive Scopes:**

```typescript
import { run, printReactiveFunction } from "babel-plugin-react-compiler";

function analyzeReactiveScopes(functionPath, config) {
  const generator = run(
    functionPath,
    config,
    "Component",
    "useMemoCache",
    null,
    "component.js",
    null
  );

  let reactiveFunction = null;

  for (const stage of generator) {
    if (stage.kind === "reactive" && stage.name.includes("Final")) {
      reactiveFunction = stage.value;
      break;
    }
  }

  if (reactiveFunction) {
    const output = printReactiveFunction(reactiveFunction);
    console.log(output);

    // Parse output to extract scope information
    const scopeMatches = output.match(/scope @\d+/g);
    console.log(`\nTotal reactive scopes: ${scopeMatches?.length || 0}`);

    return {
      scopes: reactiveFunction.scopes.length,
      dependencies: reactiveFunction.dependencies.length,
      output,
    };
  }

  return null;
}
```

---

### parseConfigPragma

Parses environment configuration from pragma strings in source comments.

```typescript { .api }
/**
 * Parses environment config from pragma string
 * @param pragma - Space-separated config flags starting with @
 * @returns Parsed environment configuration
 */
function parseConfigPragma(pragma: string): EnvironmentConfig;
```

**Usage Examples:**

```typescript
import { parseConfigPragma } from "babel-plugin-react-compiler";

// Parse single flag
const config1 = parseConfigPragma("@enableForest");
console.log(config1.enableForest); // true

// Parse multiple flags
const config2 = parseConfigPragma(
  "@enableForest @validateHooksUsage @enableMemoizationComments"
);
console.log(config2.enableForest); // true
console.log(config2.validateHooksUsage); // true
console.log(config2.enableMemoizationComments); // true

// All other options get defaults
console.log(config2.enableInstructionReordering); // true (default)
console.log(config2.validateRefAccessDuringRender); // true (default)
```

**Extracting Pragmas from Source Code:**

```typescript
import { parseConfigPragma } from "babel-plugin-react-compiler";
import * as babel from "@babel/core";

function extractAndParseConfig(code: string) {
  const ast = babel.parse(code, { sourceType: "module" });

  // Look for pragmas in comments
  const comments = ast?.comments || [];

  for (const comment of comments) {
    const text = comment.value.trim();

    // Check if comment contains pragma flags
    if (text.includes("@enable") || text.includes("@validate")) {
      // Extract just the @-prefixed flags
      const flags = text.match(/@\w+/g);

      if (flags) {
        const pragma = flags.join(" ");
        console.log("Found pragma:", pragma);

        const config = parseConfigPragma(pragma);
        return config;
      }
    }
  }

  return null;
}

// Usage
const code = `
/**
 * This component uses special compilation flags
 * @enableForest @validateHooksUsage @enableMemoizationComments
 */
function MyComponent() {
  return <div>Hello</div>;
}
`;

const config = extractAndParseConfig(code);
if (config) {
  console.log("Parsed config:", config);
}
```

**Common Pragma Flags:**

```typescript
// Validation flags
parseConfigPragma("@validateHooksUsage");
parseConfigPragma("@validateRefAccessDuringRender");
parseConfigPragma("@validateNoSetStateInRender");
parseConfigPragma("@validateMemoizedEffectDependencies");

// Optimization flags
parseConfigPragma("@enableInstructionReordering");
parseConfigPragma("@enableFunctionOutlining");
parseConfigPragma("@enableTransitivelyFreezeFunctionExpressions");

// Debugging flags
parseConfigPragma("@enableMemoizationComments");
parseConfigPragma("@disableMemoizationForDebugging");

// Feature flags
parseConfigPragma("@enableForest");
parseConfigPragma("@enableUseTypeAnnotations");
parseConfigPragma("@enablePreserveExistingMemoizationGuarantees");

// Combined flags
parseConfigPragma(
  "@validateHooksUsage @enableMemoizationComments @enableInstructionReordering"
);
```

---

### validateEnvironmentConfig

Validates and normalizes partial environment configuration, applying defaults and checking for invalid combinations.

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

type PartialEnvironmentConfig = Partial<EnvironmentConfig>;
```

**Usage Examples:**

```typescript
import {
  validateEnvironmentConfig,
  CompilerError,
  Effect,
  ValueKind,
} from "babel-plugin-react-compiler";

// Basic validation
try {
  const config = validateEnvironmentConfig({
    validateHooksUsage: true,
    enableInstructionReordering: true,
  });

  console.log("Valid config:", config);
  console.log("Defaults applied:", config.validateRefAccessDuringRender); // true
} catch (err) {
  if (err instanceof CompilerError) {
    console.error("Invalid config:", err.toString());
  }
}

// With custom hooks
const config = validateEnvironmentConfig({
  customHooks: new Map([
    [
      "useQuery",
      {
        effectKind: Effect.Read,
        valueKind: ValueKind.Mutable,
        noAlias: false,
        transitiveMixedData: true,
      },
    ],
    [
      "useMutation",
      {
        effectKind: Effect.Store,
        valueKind: ValueKind.Mutable,
        noAlias: false,
        transitiveMixedData: false,
      },
    ],
  ]),
  validateHooksUsage: true,
});

// With external functions
const config2 = validateEnvironmentConfig({
  enableEmitFreeze: {
    source: "react-compiler-runtime",
    importSpecifierName: "$freeze",
  },
  enableEmitInstrumentForget: {
    fn: {
      source: "react-compiler-runtime",
      importSpecifierName: "$instrument",
    },
    gating: {
      source: "react-compiler-runtime",
      importSpecifierName: "isInstrumentationEnabled",
    },
  },
});
```

**Validating User Input:**

```typescript
import { validateEnvironmentConfig, CompilerError } from "babel-plugin-react-compiler";

function createSafeConfig(userConfig: unknown): EnvironmentConfig | null {
  try {
    // Validate that user config is an object
    if (typeof userConfig !== "object" || userConfig === null) {
      throw new Error("Config must be an object");
    }

    // Validate with compiler
    const config = validateEnvironmentConfig(
      userConfig as PartialEnvironmentConfig
    );

    console.log("✓ Configuration validated successfully");
    return config;
  } catch (err) {
    if (err instanceof CompilerError) {
      console.error("✗ Invalid configuration:");
      for (const detail of err.details) {
        console.error(`  ${detail.reason}`);
        if (detail.description) {
          console.error(`    ${detail.description}`);
        }
      }
    } else {
      console.error("✗ Validation error:", err);
    }
    return null;
  }
}

// Usage
const userInput = {
  validateHooksUsage: true,
  enableInstructionReordering: "yes", // Invalid type
};

const config = createSafeConfig(userInput);
if (config) {
  // Use config
} else {
  // Handle validation failure
}
```

**Merging Configurations:**

```typescript
import { validateEnvironmentConfig } from "babel-plugin-react-compiler";

function mergeConfigs(
  baseConfig: PartialEnvironmentConfig,
  overrides: PartialEnvironmentConfig
): EnvironmentConfig {
  // Merge configurations
  const merged = {
    ...baseConfig,
    ...overrides,
  };

  // Validate merged result
  return validateEnvironmentConfig(merged);
}

const base = {
  validateHooksUsage: true,
  enableInstructionReordering: true,
};

const overrides = {
  enableMemoizationComments: true,
  validateHooksUsage: false, // Override
};

const final = mergeConfigs(base, overrides);
console.log(final.validateHooksUsage); // false (overridden)
console.log(final.enableMemoizationComments); // true
```

**Creating Preset Configurations:**

```typescript
import { validateEnvironmentConfig } from "babel-plugin-react-compiler";

// Development preset
function createDevConfig(): EnvironmentConfig {
  return validateEnvironmentConfig({
    // Strict validation
    validateHooksUsage: true,
    validateRefAccessDuringRender: true,
    validateNoSetStateInRender: true,
    validateMemoizedEffectDependencies: true,

    // Helpful debugging
    enableMemoizationComments: true,
    disableMemoizationForDebugging: false,

    // Conservative optimizations
    enableInstructionReordering: false,
    enableFunctionOutlining: false,
  });
}

// Production preset
function createProdConfig(): EnvironmentConfig {
  return validateEnvironmentConfig({
    // Minimal validation (assume code is correct)
    validateHooksUsage: false,
    validateRefAccessDuringRender: false,

    // No debugging
    enableMemoizationComments: false,

    // Aggressive optimizations
    enableInstructionReordering: true,
    enableFunctionOutlining: true,
    enableTransitivelyFreezeFunctionExpressions: true,
  });
}

// Usage
const devConfig = createDevConfig();
const prodConfig = createProdConfig();
```
