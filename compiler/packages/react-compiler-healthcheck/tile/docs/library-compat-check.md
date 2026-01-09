# Library Compatibility Check

Identifies third-party libraries in package.json that are known to be incompatible with the React Compiler.

## Capabilities

### Check Module Interface

The Library Compatibility check module exports an object with `run()` and `report()` methods.

```typescript { .api }
/**
 * Default export from 'react-compiler-healthcheck/src/checks/libraryCompat'
 */
interface LibraryCompatCheckModule {
  /**
   * Scans package.json files for incompatible library dependencies
   * @param source - The file content as a string (must be valid JSON)
   * @param path - The file path (must match "package.json")
   */
  run(source: string, path: string): void;

  /**
   * Outputs report of found incompatible libraries to console
   * Success: "Found no usage of incompatible libraries." (Green)
   * Failure: "Found the following incompatible libraries:" (Red) + library list
   */
  report(): void;
}
```

### run() Method

Analyzes package.json files for known incompatible library dependencies.

```typescript { .api }
/**
 * Scans package.json for known incompatible libraries
 *
 * @param source - File content as a string (JSON format)
 * @param path - File path (must end with "package.json")
 * @returns void
 *
 * Side effects:
 * - Parses JSON content
 * - Checks dependencies field against known incompatible libraries
 * - Records matches in internal Set for reporting
 */
function run(source: string, path: string): void;
```

**Behavior:**

1. **File Filtering**: Only processes files with path ending in `package.json`
2. **JSON Parsing**: Parses the source string as JSON
3. **Dependency Checking**: Examines the `dependencies` field in package.json
4. **Known Library Matching**: Compares each dependency against `config.knownIncompatibleLibraries`
5. **Result Tracking**: Adds any matches to an internal Set for deduplication

**Pattern Matching:**
- File path regex: `/package\.json$/`
- Checks only the `dependencies` field (not `devDependencies`, `peerDependencies`, etc.)

**Usage Example:**

```typescript
import libraryCompatCheck from 'react-compiler-healthcheck/src/checks/libraryCompat';

const packageJsonContent = `{
  "name": "my-app",
  "dependencies": {
    "react": "^18.0.0",
    "mobx": "^6.0.0"
  }
}`;

libraryCompatCheck.run(packageJsonContent, 'package.json');
```

### report() Method

Outputs a report of found incompatible libraries to the console.

```typescript { .api }
/**
 * Outputs incompatible library detection results to console
 * Uses colored output: green for no issues, red for found libraries
 * @returns void
 */
function report(): void;
```

**Output Format:**

If incompatible libraries found:
```
Found the following incompatible libraries:
mobx
@risingstack/react-easy-state
```
(Red text for the header, library names listed one per line)

If no incompatible libraries:
```
Found no usage of incompatible libraries.
```
(Green text)

**Usage Example:**

```typescript
import libraryCompatCheck from 'react-compiler-healthcheck/src/checks/libraryCompat';

// After running checks on package.json files
libraryCompatCheck.report();
// Output: "Found the following incompatible libraries:\nmobx" (if mobx was found)
```

### Complete Programmatic Usage

```typescript
import libraryCompatCheck from 'react-compiler-healthcheck/src/checks/libraryCompat';
import * as fs from 'fs/promises';
import { glob } from 'fast-glob';

async function checkLibraries() {
  // Find all package.json files
  const packageFiles = await glob('**/package.json', {
    ignore: ['**/node_modules/**']
  });

  for (const file of packageFiles) {
    const source = await fs.readFile(file, 'utf-8');
    libraryCompatCheck.run(source, file);
  }

  libraryCompatCheck.report();
}

checkLibraries();
```

## Configuration

### Known Incompatible Libraries

The list of incompatible libraries is defined in the `config` module.

```typescript { .api }
/**
 * Configuration object containing known incompatible libraries
 * Imported from 'react-compiler-healthcheck/src/config'
 */
export const config: {
  knownIncompatibleLibraries: string[];
};
```

**Current List:**

```typescript
config.knownIncompatibleLibraries = [
  'mobx',
  '@risingstack/react-easy-state'
];
```

**Usage Example:**

```typescript
import { config } from 'react-compiler-healthcheck/src/config';

console.log(config.knownIncompatibleLibraries);
// Output: ['mobx', '@risingstack/react-easy-state']

// You can read the list but should not modify it
// as it's used by the libraryCompat check module
```

### Why These Libraries Are Incompatible

**mobx:**
- Uses proxies and observable state that conflicts with React Compiler's automatic memoization
- The Compiler assumes immutable data patterns, but MobX mutates observable objects
- MobX's fine-grained reactivity system is incompatible with the Compiler's optimization strategy

**@risingstack/react-easy-state:**
- Built on top of MobX and inherits the same incompatibility issues
- Uses similar proxy-based observable patterns that conflict with Compiler assumptions

## Implementation Details

### State Management

The module maintains a Set to track found incompatible libraries:

```typescript
const knownIncompatibleLibrariesUsage = new Set<string>();
```

Using a Set ensures:
- Deduplication: Each library is only listed once even if found in multiple package.json files
- Efficient lookups and insertions
- Preserved insertion order for consistent reporting

### Dependency Field Checking

The check only examines the `dependencies` field in package.json:

```json
{
  "dependencies": {
    "mobx": "^6.0.0"  // ← Checked
  },
  "devDependencies": {
    "mobx": "^6.0.0"  // ← NOT checked
  },
  "peerDependencies": {
    "mobx": "^6.0.0"  // ← NOT checked
  }
}
```

**Rationale:**
- Production dependencies (`dependencies`) are the primary concern
- Dev dependencies are typically only used during development
- Peer dependencies are usually declared by libraries, not consumed directly

### JSON Parsing

The check parses JSON directly without error handling at the check level:

```typescript
const contents = JSON.parse(source);
```

**Error Behavior:**
- Invalid JSON will throw a `SyntaxError`
- Missing `dependencies` field is handled gracefully (checked with `!= null`)
- Malformed package.json files should be caught by the caller

## Dependencies

This check module requires:
- **chalk**: Terminal string styling for colored output

The config module has no dependencies.

## Notes

- The check uses exact string matching for library names (case-sensitive)
- Only runtime `dependencies` are checked, not `devDependencies` or `peerDependencies`
- The incompatible library list is maintained in a separate `config` module for easy updates
- Multiple package.json files can be processed (e.g., in a monorepo) with results deduplicated
- The Set-based tracking means finding the same library in multiple package.json files only reports it once
- The check does not validate whether the libraries are actually used in the code, only whether they're declared as dependencies
