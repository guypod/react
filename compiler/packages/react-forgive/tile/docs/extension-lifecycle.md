# Extension Lifecycle

The extension lifecycle APIs provide the entry and exit points for the React Forgive VS Code extension.

## Capabilities

### Extension Activation

Initializes the React Forgive extension when VS Code starts or when a React file is opened.

```typescript { .api }
/**
 * Activates the React Forgive extension
 * @param context - VS Code extension context for registering disposables and storing state
 */
function activate(context: vscode.ExtensionContext): void;
```

**What it does:**

1. Creates a `LanguageClient` to communicate with the LSP server
2. Configures the server module path and transport (IPC via Node)
3. Registers hover providers for JavaScript React and TypeScript React files
4. Registers a text change listener to update decorations on document changes
5. Registers the internal command `react.requestAutoDepsDecorations`
6. Starts the language client

**Server Configuration:**

The server is launched using Node.js IPC transport with the following options:

```typescript
const serverModule = context.asAbsolutePath(
  path.join('dist', 'server.js')
);

const serverOptions: ServerOptions = {
  run: { module: serverModule, transport: TransportKind.ipc },
  debug: { module: serverModule, transport: TransportKind.ipc }
};
```

**Client Configuration:**

The client is configured to handle JavaScript React and TypeScript React files:

```typescript
const clientOptions: LanguageClientOptions = {
  documentSelector: [
    { scheme: 'file', language: 'javascriptreact' },
    { scheme: 'file', language: 'typescriptreact' }
  ],
  synchronize: {
    fileEvents: workspace.createFileSystemWatcher('**/.clientrc')
  }
};
```

**Registered Providers:**

- **Hover Provider**: Registered for both `javascriptreact` and `typescriptreact` languages
- **Text Change Listener**: Updates decorations when document content changes
- **Internal Command**: `react.requestAutoDepsDecorations` - Requests decoration data from the server

**Usage Example:**

```typescript
import * as vscode from 'vscode';

// Called by VS Code when the extension activates
export function activate(context: vscode.ExtensionContext) {
  // Extension initialization logic
  // - Creates LanguageClient
  // - Registers providers and commands
  // - Starts the client
}
```

### Extension Deactivation

Cleanly shuts down the React Forgive extension when VS Code is closing or the extension is disabled.

```typescript { .api }
/**
 * Deactivates the React Forgive extension
 * @returns Promise that resolves when the client has stopped, or undefined if no client exists
 */
function deactivate(): Thenable<void> | undefined;
```

**What it does:**

1. Stops the language client gracefully if it exists
2. Cleans up resources and connections
3. Returns a promise that resolves when shutdown is complete

**Usage Example:**

```typescript
// Called by VS Code when the extension deactivates
export function deactivate(): Thenable<void> | undefined {
  if (!client) {
    return undefined;
  }
  return client.stop();
}
```

## Module Location

**File**: `client/src/extension.ts`
**Build Output**: `dist/extension.js`
**Entry Point**: Specified in `package.json` as `"main": "./dist/extension.js"`

## Related APIs

- [Auto-Dependency Decorations](./auto-deps.md) - Called from hover providers
- [LSP Server](./lsp-server.md) - The server component initialized by activate()
