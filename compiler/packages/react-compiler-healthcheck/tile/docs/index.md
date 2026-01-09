# React Compiler Healthcheck

React Compiler Healthcheck is a CLI tool that performs comprehensive health checks on React codebases to validate compliance with React's rules and assess compatibility with the React Compiler. It analyzes JavaScript and TypeScript source files to detect violations such as improper use of hooks, state mutations, and side effects that would prevent compilation optimization.

## Package Information

- **Package Name**: react-compiler-healthcheck
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: `npm install react-compiler-healthcheck`
- **Binary Command**: `react-compiler-healthcheck`

## Core Imports

This package is primarily designed as a CLI tool. Install and run directly:

```bash
npm install react-compiler-healthcheck
npx react-compiler-healthcheck
```

For advanced programmatic usage (requires importing from source):

```typescript
// Note: These imports access source files directly and are not part of the standard public API
import { config } from 'react-compiler-healthcheck/src/config';
import libraryCompatCheck from 'react-compiler-healthcheck/src/checks/libraryCompat';
import strictModeCheck from 'react-compiler-healthcheck/src/checks/strictMode';
import reactCompilerCheck from 'react-compiler-healthcheck/src/checks/reactCompiler';
```

## Basic Usage

Run the health check on your React project:

```bash
# Check all JavaScript/TypeScript files in the project
npx react-compiler-healthcheck

# Check specific files or directories
npx react-compiler-healthcheck --src "src/**/*.{js,jsx,ts,tsx}"

# The command displays as "healthcheck" in its own help/usage output
npx react-compiler-healthcheck --help
```

The tool will display a progress spinner while analyzing files and output three reports:
1. React Compiler compatibility statistics
2. StrictMode detection status
3. Incompatible library warnings

## Architecture

React Compiler Healthcheck is built around three independent health check modules:

- **CLI Entry Point** (`index.ts`): Orchestrates the health check process, handles file globbing, and manages progress display
- **Configuration** (`config.ts`): Defines known incompatible libraries and other configuration constants
- **Health Check Modules** (`checks/`): Three independent checkers that can be run programmatically:
  - **React Compiler Check**: Tests if components compile successfully with babel-plugin-react-compiler
  - **StrictMode Check**: Detects React StrictMode usage in the codebase
  - **Library Compatibility Check**: Identifies incompatible third-party libraries

Each check module follows a consistent interface with `run()` and `report()` methods, enabling both CLI and programmatic usage.

## Capabilities

### CLI Usage

Command-line interface for running health checks on React codebases with automatic file discovery and progress reporting.

```typescript { .api }
// Binary command: react-compiler-healthcheck
// Usage display: $ npx healthcheck <src>
react-compiler-healthcheck [options]

// CLI Options
interface CLIOptions {
  src?: string; // Glob pattern for source files (default: "**/+(*.{js,mjs,jsx,ts,tsx}|package.json)")
}
```

[CLI Usage](./cli-usage.md)

### Configuration

Configuration constants including the list of libraries known to be incompatible with React Compiler. Available for programmatic usage by importing from source.

```typescript { .api }
// Import from: 'react-compiler-healthcheck/src/config'
export const config: {
  knownIncompatibleLibraries: string[];
};
```

### React Compiler Compatibility Check

Tests if React components and functions can be successfully compiled with the React Compiler, identifying rule violations and incompatible patterns.

```typescript { .api }
interface CheckModule {
  run(source: string, path: string): void;
  report(): void;
}
```

[React Compiler Check](./react-compiler-check.md)

### StrictMode Detection Check

Detects React StrictMode usage in the codebase by scanning JSX files and Next.js configuration.

```typescript { .api }
interface CheckModule {
  run(source: string, path: string): void;
  report(): void;
}
```

[StrictMode Check](./strictmode-check.md)

### Library Compatibility Check

Identifies third-party libraries in package.json that are known to be incompatible with the React Compiler.

```typescript { .api }
interface CheckModule {
  run(source: string, path: string): void;
  report(): void;
}
```

[Library Compatibility Check](./library-compat-check.md)
