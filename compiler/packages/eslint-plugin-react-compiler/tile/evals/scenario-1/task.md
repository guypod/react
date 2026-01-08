# Compiler Diagnostics Monitor

Build a monitoring tool that tracks React compiler diagnostics using an ESLint plugin integration.

## Problem Description

Create a system that uses an ESLint plugin to monitor React compiler diagnostics. The tool should:
- Track compilation events through a custom logger
- Collect metrics about compilation successes and failures
- Configure which error severity levels are reported

The system should provide visibility into React compiler performance across a codebase.

## Requirements

### Configuration Setup

Create an ESLint configuration that:
- Includes the React compiler plugin
- Configures the rule with custom reportable error severity levels
- Allows specifying a custom logger for event tracking

### Logger Implementation

Implement a logger that:
- Receives compilation events (successes, errors, diagnostics)
- Stores event data including filenames and event details
- Provides methods to retrieve all captured events
- Calculates statistics (total compilations, success count, error count)

### Event Types to Handle

- Compilation successes with memoization metrics (slots, blocks, values)
- Compilation errors with severity levels (InvalidReact, InvalidJS, etc.)
- File and location information for each event

## Test Cases

### Configuration Test

- Creates an ESLint configuration with the plugin and rule settings [@test](./test/config.test.js)

### Logger Test

- Logger captures and stores compilation events correctly [@test](./test/logger.test.js)

### Statistics Test

- Calculates compilation statistics from captured events [@test](./test/stats.test.js)

## Implementation

[@generates](./src/index.js)

## API

```javascript { #api }
// Create an ESLint configuration that includes the compiler plugin
export function createCompilerConfig(options);

// Logger implementation for tracking compilation events
export class CompilationLogger {
  constructor();
  logEvent(filename, event);
  getEvents();
  getStats();
}

// Get error severity constants
export function getErrorSeverities();
```

## Dependencies { .dependencies }

### eslint-plugin-react-compiler { .dependency }

Provides React compiler integration and diagnostics reporting.

### eslint { .dependency }

Core ESLint functionality for linting configuration.
