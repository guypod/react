# StrictMode Check

Detects React StrictMode usage in the codebase by scanning JSX files for StrictMode components and Next.js configuration files for the reactStrictMode flag.

## Capabilities

### Check Module Interface

The StrictMode check module exports an object with `run()` and `report()` methods.

```typescript { .api }
/**
 * Default export from 'react-compiler-healthcheck/src/checks/strictMode'
 */
interface StrictModeCheckModule {
  /**
   * Scans a source file for StrictMode usage
   * @param source - The file content as a string
   * @param path - The file path
   */
  run(source: string, path: string): void;

  /**
   * Outputs StrictMode detection status to console
   * Success: "StrictMode usage found." (Green)
   * Failure: "StrictMode usage not found." (Red)
   */
  report(): void;
}
```

### run() Method

Scans source files for React StrictMode usage patterns.

```typescript { .api }
/**
 * Detects React StrictMode usage in source files
 *
 * @param source - File content as a string
 * @param path - File path
 * @returns void
 *
 * Side effects:
 * - Updates internal boolean flag if StrictMode is detected
 * - Short-circuits if StrictMode already found in previous files
 */
function run(source: string, path: string): void;
```

**Behavior:**

1. **Early Exit**: If StrictMode has already been found in a previous file, the method returns immediately without processing
2. **Next.js Config Detection**: If the file path ends with `next.config.js` or `next.config.mjs`, searches for `reactStrictMode: true` pattern
3. **JSX File Detection**: If the file has `.js`, `.ts`, `.jsx`, or `.tsx` extension, searches for `<React.StrictMode>` or `<StrictMode>` JSX tags
4. **State Update**: Sets internal flag to `true` if StrictMode is detected

**Detection Patterns:**

Next.js configuration files (`next.config.js`, `next.config.mjs`):
```javascript
// Pattern: reactStrictMode: true
const nextConfig = {
  reactStrictMode: true,
  // ... other config
};
```

JSX files (`.js`, `.ts`, `.jsx`, `.tsx`):
```jsx
// Pattern: <React.StrictMode> or <StrictMode>
import React from 'react';

function App() {
  return (
    <React.StrictMode>
      <MyApp />
    </React.StrictMode>
  );
}
```

**Regular Expressions:**

- Next.js config file: `/^next\.config\.(js|mjs)$/`
- JSX file extensions: `/(js|ts|jsx|tsx)$/`
- StrictMode JSX tag: `/<(React\.StrictMode|StrictMode)>/`
- Next.js StrictMode config: `/reactStrictMode:\s*true/`

**Usage Example:**

```typescript
import strictModeCheck from 'react-compiler-healthcheck/src/checks/strictMode';

// Check JSX file
const jsxSource = `
import React from 'react';
export default function App() {
  return <React.StrictMode><MyComponent /></React.StrictMode>;
}
`;
strictModeCheck.run(jsxSource, 'App.tsx');

// Check Next.js config
const nextConfig = `
module.exports = {
  reactStrictMode: true,
  swcMinify: true,
};
`;
strictModeCheck.run(nextConfig, 'next.config.js');
```

### report() Method

Outputs StrictMode detection status to the console.

```typescript { .api }
/**
 * Outputs StrictMode detection result to console
 * Uses colored output: green for found, red for not found
 * @returns void
 */
function report(): void;
```

**Output Format:**

If StrictMode found:
```
StrictMode usage found.
```
(Green text)

If StrictMode not found:
```
StrictMode usage not found.
```
(Red text)

**Usage Example:**

```typescript
import strictModeCheck from 'react-compiler-healthcheck/src/checks/strictMode';

// After running checks on multiple files
strictModeCheck.report();
// Output: "StrictMode usage found." (if detected in any file)
```

### Complete Programmatic Usage

```typescript
import strictModeCheck from 'react-compiler-healthcheck/src/checks/strictMode';
import * as fs from 'fs/promises';
import { glob } from 'fast-glob';

async function checkStrictMode() {
  // Check all source files
  const sourceFiles = await glob('src/**/*.{js,jsx,ts,tsx}', {
    ignore: ['**/node_modules/**', '**/__tests__/**']
  });

  for (const file of sourceFiles) {
    const source = await fs.readFile(file, 'utf-8');
    strictModeCheck.run(source, file);
  }

  // Check Next.js config if it exists
  try {
    const nextConfig = await fs.readFile('next.config.js', 'utf-8');
    strictModeCheck.run(nextConfig, 'next.config.js');
  } catch {
    // No Next.js config found
  }

  strictModeCheck.report();
}

checkStrictMode();
```

## Implementation Details

### State Management

The module maintains a single boolean flag that tracks whether StrictMode has been detected:

```typescript
let StrictModeUsage = false;
```

Once set to `true`, it remains `true` for the lifetime of the module, and subsequent `run()` calls short-circuit immediately.

### Why StrictMode Matters

React StrictMode is important for React Compiler compatibility because:
1. It helps identify unsafe lifecycle methods and legacy patterns
2. It detects potential side effects in render functions
3. It warns about deprecated APIs that may conflict with compiler optimizations
4. It enables additional development-time checks that surface issues the compiler might encounter

### Detection Limitations

**What the check detects:**
- Direct usage of `<React.StrictMode>` or `<StrictMode>` JSX tags in any source file
- Next.js `reactStrictMode: true` configuration option

**What the check does NOT detect:**
- StrictMode imported with different names (e.g., `import { StrictMode as SM }`)
- Dynamic StrictMode usage (e.g., conditionally rendered)
- StrictMode enabled through other framework configurations
- StrictMode in files excluded by ignore patterns

## Dependencies

This check module requires:
- **chalk**: Terminal string styling for colored output

No parsing dependencies are needed as the check uses simple regex pattern matching.

## Notes

- The check uses regex pattern matching rather than AST parsing for performance
- Once StrictMode is found in any file, subsequent files are skipped for efficiency
- The check is case-sensitive (e.g., `strictMode` will not match)
- Whitespace around `reactStrictMode: true` is flexible due to the `\s*` regex pattern
- The module state is global - calling `run()` accumulates state across all invocations until process exit
