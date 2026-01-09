# React Hooks Usage Analyzer

Build a check module for the react-compiler-healthcheck tool that tracks React Hook usage across a codebase.

## Capabilities

### File Processing

The module processes JavaScript and TypeScript files to detect React Hook usage.

- Processes files with extensions .js, .jsx, .ts, .tsx [@test](./src/hooksCheck.test.ts)
- Ignores files with other extensions [@test](./src/hooksCheck.test.ts)

### Hook Detection

The module detects React Hook function calls in source code.

- Detects built-in React Hooks like useState, useEffect, useContext [@test](./src/hooksCheck.test.ts)
- Detects custom Hooks following the "use" naming convention [@test](./src/hooksCheck.test.ts)
- Counts each occurrence of Hook usage across all files [@test](./src/hooksCheck.test.ts)

Hooks are identified using this pattern:
- Function names starting with "use" followed by an uppercase letter
- Called as functions (followed by opening parenthesis)
- Detected using regular expressions

### Usage Reporting

The module outputs a summary of Hook usage statistics.

- Reports the total number of files containing Hooks [@test](./src/hooksCheck.test.ts)
- Lists each Hook name with its total count across all files [@test](./src/hooksCheck.test.ts)
- Displays count in green if hooks found, red if none found [@test](./src/hooksCheck.test.ts)

Example output:
```
Found Hooks in 5 files:
  useState: 12
  useEffect: 8
  useContext: 3
```

## Implementation

[@generates](./src/hooksCheck.ts)

## API

```typescript { #api }
interface CheckModule {
  run(source: string, path: string): void;
  report(): void;
}

export default CheckModule;
```

## Dependencies { .dependencies }

### chalk { .dependency }

Provides terminal string styling for colored output.

### fast-glob { .dependency }

Enables file pattern matching for the CLI tool.
