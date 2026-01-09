# CLI Usage

Command-line interface for running React Compiler health checks on your codebase.

## Capabilities

### Binary Command

The package provides a CLI binary that can be executed via npx or after installation.

```typescript { .api }
/**
 * Binary command: react-compiler-healthcheck
 * Display name in usage/help: healthcheck
 *
 * The binary is named "react-compiler-healthcheck" but the tool
 * displays itself as "healthcheck" in its own help and usage output.
 */
```

**Installation:**

```bash
npm install react-compiler-healthcheck
```

**Execution:**

```bash
# Run with npx (no installation required)
npx react-compiler-healthcheck

# After installation, run directly
react-compiler-healthcheck

# The command identifies itself as "healthcheck" in its help output
npx react-compiler-healthcheck --help
# Output: $ npx healthcheck <src>
```

### CLI Options

#### --src Option

Specifies a glob pattern for matching source files to analyze.

```typescript { .api }
/**
 * @option --src
 * @type string
 * @default "**/+(*.{js,mjs,jsx,ts,tsx}|package.json)"
 * @description Glob expression matching source files to compile
 */
```

**Usage Examples:**

```bash
# Use default pattern (all JS/TS files and package.json)
npx react-compiler-healthcheck

# Check specific directory
npx react-compiler-healthcheck --src "src/**/*.{js,jsx,ts,tsx}"

# Check multiple patterns
npx react-compiler-healthcheck --src "src/**/*.tsx"

# Check all files in current directory
npx react-compiler-healthcheck --src "./**/*.{js,jsx}"
```

### Automatic File Exclusions

The CLI automatically ignores common directories that should not be analyzed:

```typescript { .api }
/**
 * Automatic ignore patterns:
 * - "**/node_modules/**"
 * - "**/dist/**"
 * - "**/tests/**"
 * - "**/__tests__/**"
 * - "**/__mocks__/**"
 * - "**/__e2e__/**"
 */
```

These patterns are always excluded regardless of the `--src` option value.

### Supported File Types

The health check processes the following file types:

- **JavaScript**: `.js`, `.mjs`
- **TypeScript**: `.ts`, `.tsx`
- **JSX**: `.jsx`
- **Configuration**: `package.json`, `next.config.js`, `next.config.mjs`

**File Processing Behavior:**

- `.js`, `.mjs`, `.jsx`, `.ts`, `.tsx`: Analyzed by React Compiler check and StrictMode check
- `package.json`: Analyzed by Library Compatibility check
- `next.config.js`, `next.config.mjs`: Analyzed by StrictMode check

### Progress Indication

The CLI displays a spinner during analysis showing the currently processed file:

```bash
⠹ Checking src/components/MyComponent.tsx
```

### Output Reports

After analyzing all files, the CLI outputs three reports:

1. **React Compiler Compatibility Report**
   - Shows number of successfully compiled components
   - Format: "Successfully compiled X out of Y components."
   - Color: Green

2. **StrictMode Detection Report**
   - Indicates whether StrictMode usage was found
   - Success: "StrictMode usage found." (Green)
   - Failure: "StrictMode usage not found." (Red)

3. **Library Compatibility Report**
   - Lists any incompatible libraries found in dependencies
   - Success: "Found no usage of incompatible libraries." (Green)
   - Failure: "Found the following incompatible libraries:" (Red) followed by library names

**Example Output:**

```bash
✔ Successfully compiled 42 out of 45 components.
✔ StrictMode usage found.
✔ Found no usage of incompatible libraries.
```

### Exit Behavior

The CLI runs asynchronously and completes after all reports are displayed. It does not return specific exit codes for success or failure.

### Complete Usage Example

```bash
# Full workflow
cd my-react-project
npm install react-compiler-healthcheck
npx react-compiler-healthcheck --src "src/**/*.{ts,tsx}"
```

This will:
1. Find all TypeScript and TSX files in the `src/` directory
2. Exclude test directories and node_modules
3. Run three health checks on each file
4. Display progress with a spinner
5. Output three colored reports with results
