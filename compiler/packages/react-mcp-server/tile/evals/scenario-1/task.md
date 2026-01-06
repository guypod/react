# React Performance Optimizer

Build a React component performance analyzer that uses advanced compilation and runtime measurement to identify and report optimization opportunities.

## Problem Statement

You need to create a tool that analyzes React components to determine if they can benefit from automatic memoization. The tool should:

1. Compile React component code and extract compilation diagnostics
2. Identify whether the component compiled successfully or encountered bailouts
3. When compilation succeeds, verify the optimization by checking for memoization cache initialization
4. For components with bailouts, extract the specific error messages and their locations

## Requirements

### Component Analyzer

Create a TypeScript file `analyzer.ts` that exports an `analyzeComponent` function with the following signature:

```typescript
interface CompilationResult {
  success: boolean;
  hasMemoization: boolean;
  cacheSize: number | null;
  errors: Array<{
    message: string;
    location: string | null;
  }>;
}

async function analyzeComponent(code: string): Promise<CompilationResult>
```

The function should:

1. **Compile the component code** using the compilation functionality
   - Use appropriate presets for TypeScript and JSX parsing
   - Configure the compiler to capture diagnostic messages

2. **Analyze the compiled output** when compilation succeeds:
   - Check if the output includes memoization imports (the runtime cache helper)
   - Extract the cache size from the cache initialization constant if present
   - A cache size of 0 or null indicates no memoization was applied

3. **Extract error information** when compilation fails or bails out:
   - Collect all bailout and error messages
   - Include source code locations when available
   - Format locations as "line:column" or null if unavailable

4. **Return a result object** with:
   - `success`: true if code compiled without errors/bailouts
   - `hasMemoization`: true if the compiler added memoization (cache size > 0)
   - `cacheSize`: the number of memoization slots (null if no memoization)
   - `errors`: array of error objects with messages and locations

### Test Cases

- Given a simple pure component, the function returns success=true with memoization applied [@test](../test/analyzer.test.ts)
- Given a component that mutates state directly, the function returns success=false with a bailout error [@test](../test/analyzer.test.ts)
- Given a component with existing manual memoization, the function returns success=false with a specific bailout about manual memoization [@test](../test/analyzer.test.ts)
- Given valid code with no optimization opportunities, the function returns success=true with hasMemoization=false and cacheSize=null [@test](../test/analyzer.test.ts)

## Implementation

[@generates](./src/analyzer.ts)

## API

```typescript { #api }
export interface CompilationResult {
  success: boolean;
  hasMemoization: boolean;
  cacheSize: number | null;
  errors: Array<{
    message: string;
    location: string | null;
  }>;
}

export async function analyzeComponent(code: string): Promise<CompilationResult>;
```

## Dependencies { .dependencies }

### @babel/core { .dependency }

Provides core Babel transformation capabilities for parsing and transforming JavaScript/TypeScript code.

[@satisfied-by](@babel/core)

### @babel/parser { .dependency }

Provides parsing capabilities to convert source code into an Abstract Syntax Tree (AST).

[@satisfied-by](@babel/parser)

### babel-plugin-react-compiler { .dependency }

The React Compiler plugin that automatically optimizes React components by inserting memoization.

[@satisfied-by](babel-plugin-react-compiler)
