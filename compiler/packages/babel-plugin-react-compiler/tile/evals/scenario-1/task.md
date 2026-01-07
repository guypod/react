# React Compiler Configuration with Custom Validation

## Overview

Configure the React Compiler plugin for a React application with specific requirements for custom hooks, validation rules, and feature-flagged rollout.

## Requirements

### 1. Configure Custom Hook Behavior

Your application uses a custom hook called `useDataFetcher` that fetches and caches data. The compiler needs to understand this hook's behavior:

- The hook reads from its arguments but does not mutate them
- The hook returns non-constant (mutable) values
- Arguments passed to the hook should not be aliased
- The return value contains JSON-serializable data

Configure the compiler to recognize this custom hook with the appropriate effect and value semantics.

### 2. Enable Strict Validation Rules

Enable the following validation checks to enforce React best practices:

- Validate that refs are not accessed during render (ref.current should not be read in component bodies)
- Validate that effect dependencies are properly memoized
- Ensure that capitalized functions are not called unsafely (except for allowed components: `App`, `Header`, `Footer`)

### 3. Implement Feature Gating

The team wants to gradually roll out the compiled code using a feature flag. Configure the plugin to:

- Generate both compiled and uncompiled versions of functions
- Import a feature flag function from the module `@company/feature-flags`
- Use the named export `isCompilerEnabled` as the gating function
- The compiled version should only run when the feature flag returns true

### 4. Set Compilation Mode and Error Handling

- Use "infer" mode to automatically detect components and hooks
- Configure the compiler to continue compilation even if errors occur (do not throw exceptions on errors)

## Implementation

Create a complete configuration file that satisfies all the requirements above. Your solution should be a JavaScript module that exports the plugin configuration.

## File Structure

```
src/
  babel.config.js     # Your Babel configuration file
```

## Dependencies { .dependencies }

### babel-plugin-react-compiler { .dependency }

Babel plugin that optimizes React applications by automatically handling memoization and minimizing re-renders.

## Test Cases

### Test Case 1: Configuration Exports { .test }

**Input**: Load the babel.config.js file

**Expected Output**: The file should export a valid Babel configuration object with the React Compiler plugin configured

### Test Case 2: Custom Hook Configuration { .test }

**Input**: Check the custom hooks configuration in the plugin options

**Expected Output**: The configuration should include a custom hook named `useDataFetcher` with appropriate effect and value settings

### Test Case 3: Validation Rules { .test }

**Input**: Check the environment configuration for validation flags

**Expected Output**: The configuration should enable validation for ref access during render, memoized effect dependencies, and capitalized calls with an allowlist

### Test Case 4: Feature Gating Setup { .test }

**Input**: Check the gating configuration in the plugin options

**Expected Output**: The configuration should include a gating object that imports `isCompilerEnabled` from `@company/feature-flags`
