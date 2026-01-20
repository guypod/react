# Code Compilation

Integration with Babel and React Compiler for analyzing React code. The compilation module transforms source code and formats output with Prettier.

## Capabilities

### Compile Function

Main compilation function that analyzes React code using Babel and the React Compiler plugin.

```typescript { .api }
/**
 * Compiles React source code using Babel and React Compiler
 * @param options - Compilation options
 * @returns Babel compilation result or null on error
 */
async function compile(options: CompileOptions): Promise<BabelCore.BabelFileResult | null>;

type CompileOptions = {
  /** Source code to compile */
  text: string;
  /** File path for the source code */
  file: string;
  /** React Compiler plugin options */
  options: PluginOptions | null;
};
```

**What it does:**

1. Parses the source code with Babel parser
   - Supports TypeScript and JSX syntax
   - Uses module source type
2. Transforms the code with React Compiler plugin (babel-plugin-react-compiler)
3. Formats the output with Prettier (semi=false)
4. Stores the result in `lastResult` for retrieval
5. Returns the BabelFileResult containing compiled code

**Babel Configuration:**

- **Parser Plugins**: `['typescript', 'jsx']`
- **Source Type**: `'module'`
- **No Config Files**: `babelrc: false`, `configFile: false`
- **Prettier Formatting**: Enabled with `semi: false`

**Usage Example:**

```typescript
import { compile } from './compiler';

const result = await compile({
  text: `
    import { useEffect, useState } from 'react';

    function MyComponent() {
      const [count, setCount] = useState(0);

      useEffect(() => {
        console.log(count);
      });

      return <div>{count}</div>;
    }
  `,
  file: 'MyComponent.tsx',
  options: compilerOptions
});

if (result) {
  console.log('Compiled code:', result.code);
}
```

### Last Result Variable

Global reference to the last successful compilation result.

```typescript { .api }
/**
 * Global reference to the last successful Babel compilation result
 */
let lastResult: BabelCore.BabelFileResult | null;

interface BabelFileResult {
  /** Compiled code as a string */
  code: string | null;
  /** Source map for the compilation */
  map: any | null;
  /** Babel AST */
  ast: any | null;
  /** Metadata from the compilation */
  metadata?: any;
}
```

**Usage Example:**

```typescript
import { lastResult } from './compiler';

// After calling compile(), access the last result
if (lastResult) {
  console.log('Last compiled code:', lastResult.code);
}
```

## Babel-to-VSCode Conversion Utilities

### Babel Location to Range

Converts Babel SourceLocation to VS Code Range format.

```typescript { .api }
/**
 * Converts Babel SourceLocation to VS Code LSP Range
 * @param loc - Babel SourceLocation with line and column information
 * @returns VS Code Range with adjusted line numbers (0-indexed), or null for symbol locations
 */
function babelLocationToRange(loc: SourceLocation): Range | null;

interface SourceLocation {
  start: {
    line: number;    // Babel uses 1-indexed lines
    column: number;
  };
  end: {
    line: number;
    column: number;
  };
}

interface Range {
  start: Position;
  end: Position;
}

interface Position {
  line: number;      // VS Code uses 0-indexed lines
  character: number;
}
```

**What it does:**

1. Adjusts line numbers from Babel's 1-indexed format to VS Code's 0-indexed format
2. Returns null for symbol locations (identifiers without meaningful source locations)
3. Creates a VS Code Range object with start and end positions

**Usage Example:**

```typescript
import { babelLocationToRange } from './compiler/compat';

// Convert Babel location to VS Code range
const babelLoc = {
  start: { line: 10, column: 4 },
  end: { line: 10, column: 20 }
};

const vscodeRange = babelLocationToRange(babelLoc);
// Result: { start: { line: 9, character: 4 }, end: { line: 9, character: 20 } }
```

### Get Range First Character

Refines a range to only the first character, used for precise code lens positioning.

```typescript { .api }
/**
 * Refines a range to only the first character
 * @param range - VS Code Range to refine
 * @returns New Range containing only the first character
 */
function getRangeFirstCharacter(range: Range): Range;
```

**What it does:**

1. Takes a VS Code Range
2. Returns a new Range with the same start position
3. Sets the end position to be one character after the start

**Usage Example:**

```typescript
import { getRangeFirstCharacter } from './compiler/compat';

const range = {
  start: { line: 5, character: 10 },
  end: { line: 5, character: 30 }
};

const firstChar = getRangeFirstCharacter(range);
// Result: { start: { line: 5, character: 10 }, end: { line: 5, character: 11 } }
```

## Compiler Event Logging

The React Compiler emits events during compilation that are captured by a logger:

### Logger Configuration

```typescript { .api }
interface Logger {
  /**
   * Logs and processes compiler events
   * @param filename - Source file name or null
   * @param event - Compiler event (CompileSuccess, AutoDepsDecorations, AutoDepsEligible, etc.)
   */
  logEvent(filename: string | null, event: LoggerEvent): void;
}
```

**Logged Events:**

1. **CompileSuccess** - Tracked in `compiledFns` set for code lenses
2. **AutoDepsDecorations** - Stored in `autoDepsDecorations` array for decoration requests
3. **AutoDepsEligible** - Used to generate code actions for applying inferred dependencies

## Dependencies

### Babel Dependencies

- **@babel/core** ^7.26.0 - JavaScript/TypeScript parser and transformer
- **@babel/parser** ^7.26.0 - Babel parser
- **@babel/plugin-syntax-typescript** ^7.25.9 - TypeScript syntax support
- **@babel/types** ^7.26.0 - Babel AST types

### React Compiler

- **babel-plugin-react-compiler** - React Compiler Babel plugin

### Code Formatting

- **prettier** ^3.3.3 - Code formatter
  - Plugins: babel, estree, typescript
  - Configuration: `semi: false`

## Module Locations

**Files**:
- `server/src/compiler/index.ts` - Main compilation logic
- `server/src/compiler/compat.ts` - Babel-to-VSCode conversion utilities

## Related APIs

- [LSP Server](./lsp-server.md) - Uses compile() to analyze React code on document changes
- [Utilities](./utilities.md) - Additional range and position utilities
