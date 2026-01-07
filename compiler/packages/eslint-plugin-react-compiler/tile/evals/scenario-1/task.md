# React Compiler Lint Configuration

Build a custom ESLint configuration module that leverages advanced features of the React compiler plugin to provide detailed compilation metrics and configurable error reporting for a React application.

## Objective

Create a configuration module that integrates the React compiler ESLint plugin with custom logging, selective error severity reporting, and compilation success metrics tracking.

## Requirements

### Core Functionality

Your module should export a function `createReactCompilerConfig` that accepts configuration options and returns an ESLint rule configuration object.

#### Configuration Options

The function should accept an options object with the following properties:

- `severityLevels`: An array of strings specifying which error severity levels to report (e.g., `['InvalidReact', 'InvalidJS']`)
- `enableMetrics`: A boolean indicating whether to track compilation success metrics
- `onCompilationComplete`: An optional callback function that receives compilation results

#### Metrics Tracking

When `enableMetrics` is true, the module should:

- Track successful compilations
- Record the number of memoized values for each successful compilation
- Record the function or component name being compiled
- Pass this information to the `onCompilationComplete` callback if provided

#### Error Reporting

The module should:

- Convert the severity level strings to the appropriate internal format
- Configure the plugin to report only the specified severity levels
- Handle errors gracefully if invalid severity levels are provided

### Test Cases

- Creates configuration with default severity levels [@test](./config.test.js)
- Creates configuration with custom severity levels [@test](./config.test.js)
- Tracks compilation metrics when enabled [@test](./config.test.js)
- Invokes callback with compilation results [@test](./config.test.js)

## Implementation

[@generates](./src/config.js)

## API

```javascript { #api }
/**
 * Creates an ESLint rule configuration for the React compiler plugin
 * @param {Object} options - Configuration options
 * @param {string[]} options.severityLevels - Array of error severity level names
 * @param {boolean} options.enableMetrics - Whether to track compilation metrics
 * @param {Function} options.onCompilationComplete - Callback for compilation results
 * @returns {Array} ESLint rule configuration array [severity, options]
 */
export function createReactCompilerConfig(options);
```

## Dependencies { .dependencies }

### eslint-plugin-react-compiler { .dependency }

Provides React compiler integration for ESLint.

This package is available from npm: `eslint-plugin-react-compiler`
