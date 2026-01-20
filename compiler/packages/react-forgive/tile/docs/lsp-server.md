# Language Server Protocol Implementation

The LSP server performs React code analysis using Babel and the React Compiler. It provides code lenses, code actions, and handles custom requests for auto-dependency decorations.

## Server Initialization

### Connection and Document Management

```typescript { .api }
/**
 * LSP connection with all proposed features enabled
 */
const connection: Connection;

/**
 * Text document manager for tracking open documents
 */
const documents: TextDocuments<TextDocument>;
```

**Usage Example:**

```typescript
import { createConnection, ProposedFeatures } from 'vscode-languageserver/node';
import { TextDocuments } from 'vscode-languageserver';
import { TextDocument } from 'vscode-languageserver-textdocument';

const connection = createConnection(ProposedFeatures.all);
const documents = new TextDocuments(TextDocument);

// Make the text document manager listen on the connection
documents.listen(connection);

// Start listening for messages
connection.listen();
```

## Server Capabilities

### Initialize Handler

Initializes the LSP server and returns server capabilities.

```typescript { .api }
/**
 * Handles initialization request from the client
 * @param params - Initialization parameters from the client
 * @returns Server capabilities and initialization result
 */
function onInitialize(params: InitializeParams): InitializeResult;

interface InitializeResult {
  capabilities: ServerCapabilities;
}

interface ServerCapabilities {
  /** Full document synchronization */
  textDocumentSync: TextDocumentSyncKind.Full;
  /** Code lens support with resolve capability */
  codeLensProvider: {
    resolveProvider: boolean;
  };
  /** Code action support with resolve capability */
  codeActionProvider: {
    resolveProvider: boolean;
  };
}
```

**What it does:**

1. Initializes React Compiler with default options
2. Configures effect dependency inference for:
   - `React.useEffect` (autodepsIndex: 1)
   - `shared-runtime.useSpecialEffect` (autodepsIndex: 2)
   - `useEffectWrapper` default export (autodepsIndex: 1)
3. Returns server capabilities (text sync, code lens, code actions)

**Usage Example:**

```typescript
connection.onInitialize((params: InitializeParams) => {
  // Initialize compiler options
  compilerOptions = /* ... */;

  return {
    capabilities: {
      textDocumentSync: TextDocumentSyncKind.Full,
      codeLensProvider: { resolveProvider: true },
      codeActionProvider: { resolveProvider: true }
    }
  };
});
```

### Initialized Handler

Called after initialization completes.

```typescript { .api }
/**
 * Handles initialized notification from the client
 */
function onInitialized(): void;
```

## Document Change Handling

### Document Content Changes

```typescript { .api }
/**
 * Handles document content changes
 * @param event - Text document change event containing the updated document
 */
function onDidChangeContent(event: TextDocumentChangeEvent<TextDocument>): Promise<void>;
```

**What it does:**

1. Resets compiler state (clears compiledFns, autoDepsDecorations, codeActionEvents)
2. Gets the document URI and text content
3. Calls `compile()` to analyze the React code
4. Logs compilation errors if any occur

**Usage Example:**

```typescript
documents.onDidChangeContent(async (event) => {
  const document = event.document;

  // Reset state
  compiledFns.clear();
  autoDepsDecorations = [];
  codeActionEvents = [];

  // Compile the document
  await compile({
    text: document.getText(),
    file: document.uri,
    options: compilerOptions
  });
});
```

### File System Changes

```typescript { .api }
/**
 * Handles file system changes
 * @param change - File change event
 */
function onDidChangeWatchedFiles(change: DidChangeWatchedFilesParams): void;
```

**What it does:**

1. Resets compiler tracking state
2. Clears compiledFns, autoDepsDecorations, and codeActionEvents

## Code Lens Support

### Code Lens Provider

```typescript { .api }
/**
 * Provides code lenses for compiled functions
 * @param params - Code lens request parameters containing document URI
 * @returns Array of code lenses or null if no functions compiled
 */
function onCodeLens(params: CodeLensParams): CodeLens[] | null;

interface CodeLens {
  /** Range where the code lens should appear */
  range: Range;
  /** Command to execute when code lens is clicked (optional) */
  command?: Command;
  /** Data to be passed to code lens resolve (optional) */
  data?: any;
}

interface Range {
  start: Position;
  end: Position;
}

interface Position {
  line: number;
  character: number;
}
```

**What it does:**

1. Iterates over all successfully compiled functions
2. Creates a code lens for each function at the first character of its range
3. Sets the title to "Optimized by React Compiler"
4. Returns null if no functions were compiled

**Usage Example:**

```typescript
connection.onCodeLens((params) => {
  if (compiledFns.size === 0) {
    return null;
  }

  const lenses: CodeLens[] = [];
  for (const event of compiledFns) {
    if (event.fnLoc) {
      const range = getRangeFirstCharacter(babelLocationToRange(event.fnLoc));
      lenses.push({
        range,
        command: {
          title: 'Optimized by React Compiler',
          command: ''
        }
      });
    }
  }
  return lenses;
});
```

### Code Lens Resolve

```typescript { .api }
/**
 * Resolves a code lens with additional data
 * @param lens - Code lens to resolve
 * @returns Resolved code lens with compiled code output
 */
function onCodeLensResolve(lens: CodeLens): CodeLens;
```

**What it does:**

1. Returns the code lens as-is (resolve logic can add compiled code details)
2. Can include the compiled output in the lens data

## Code Action Support

### Code Action Provider

```typescript { .api }
/**
 * Provides code actions (quick fixes) for auto-dependency eligible functions
 * @param params - Code action request parameters containing document URI and range
 * @returns Array of code actions
 */
function onCodeAction(params: CodeActionParams): CodeAction[];

interface CodeAction {
  /** Title displayed to the user */
  title: string;
  /** Code action kind (e.g., quickfix, refactor) */
  kind: CodeActionKind;
  /** Edit to apply when code action is selected */
  edit?: WorkspaceEdit;
  /** Command to execute when code action is selected (alternative to edit) */
  command?: Command;
  /** Data to be passed to code action resolve */
  data?: any;
}

interface WorkspaceEdit {
  /** Map of document URI to text edits */
  changes?: { [uri: string]: TextEdit[] };
}

interface TextEdit {
  /** Range to replace */
  range: Range;
  /** New text to insert */
  newText: string;
}
```

**What it does:**

1. Filters code action events to find those within the requested range
2. Creates a code action for each eligible function
3. Generates a WorkspaceEdit with the inferred dependency array text
4. Triggers re-decoration after applying the code action

**Usage Example:**

```typescript
connection.onCodeAction((params) => {
  const actions: CodeAction[] = [];

  for (const event of codeActionEvents) {
    if (isRangeWithinRange(event.anchorRange, params.range)) {
      actions.push({
        title: event.title,
        kind: event.kind,
        edit: {
          changes: {
            [params.textDocument.uri]: [{
              range: event.editRange,
              newText: event.newText
            }]
          }
        }
      });
    }
  }

  return actions;
});
```

## Custom Request Handler

### Auto-Deps Decorations Request

```typescript { .api }
/**
 * Handles custom 'react/autodeps_decorations' request
 * @param params - Request parameters containing position to check
 * @returns Decoration data if position is in an eligible function, null otherwise
 */
function onRequest(
  type: typeof AutoDepsDecorationsRequest.type,
  handler: (params: AutoDepsDecorationsParams) => Promise<AutoDepsDecorationsLSPEvent | null>
): void;

interface AutoDepsDecorationsParams {
  position: Position;
}

interface AutoDepsDecorationsLSPEvent {
  useEffectCallExpr: Range;
  decorations: Array<Range>;
}
```

**What it does:**

1. Receives a position from the client
2. Checks if the position falls within any auto-dependency eligible function
3. Returns decoration data (useEffect range and inferred dependencies) if found
4. Returns null if position is not in an eligible function

**Usage Example:**

```typescript
import { AutoDepsDecorationsRequest } from './requests/autodepsdecorations';

connection.onRequest(
  AutoDepsDecorationsRequest.type,
  async (params) => {
    for (const event of autoDepsDecorations) {
      if (isPositionWithinRange(params.position, event.useEffectCallExpr)) {
        return mapCompilerEventToLSPEvent(event);
      }
    }
    return null;
  }
);
```

## Server State

### Compiler Options

```typescript { .api }
/**
 * React Compiler plugin configuration options
 */
let compilerOptions: PluginOptions | null;
```

### Compiled Functions

```typescript { .api }
/**
 * Set of successfully compiled functions
 */
let compiledFns: Set<CompileSuccessEvent>;

interface CompileSuccessEvent {
  fnLoc: SourceLocation | null;
  fnName: string;
  // ... other compiler event properties
}
```

### Auto-Deps Decorations

```typescript { .api }
/**
 * Array of auto-dependency decoration events
 */
let autoDepsDecorations: Array<AutoDepsDecorationsLSPEvent>;
```

### Code Action Events

```typescript { .api }
/**
 * Array of code action events for quick fixes
 */
let codeActionEvents: Array<CodeActionLSPEvent>;

type CodeActionLSPEvent = {
  title: string;
  kind: CodeActionKind;
  newText: string;
  anchorRange: Range;
  editRange: { start: Position; end: Position };
};
```

## Supported Languages

```typescript { .api }
/**
 * Set of supported language IDs
 */
const SUPPORTED_LANGUAGE_IDS: Set<string>;
```

**Languages:**
- `javascript`
- `javascriptreact`
- `typescript`
- `typescriptreact`

## Module Location

**File**: `server/src/index.ts`
**Build Output**: `dist/server.js`

## Related APIs

- [Compilation](./compilation.md) - Called by onDidChangeContent to analyze React code
- [Auto-Dependency Decorations](./auto-deps.md) - Client-side counterpart that requests decorations
- [Utilities](./utilities.md) - Range and position utilities used by event handlers
