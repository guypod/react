# Code Analysis Extension

A Visual Studio Code extension that provides real-time code quality suggestions through a language server. The extension communicates with a language server to analyze code and display inline suggestions.

## Capabilities

### Extension activation and client setup

The extension activates for JavaScript and TypeScript files and creates a client that communicates with a language server using inter-process communication.

- The extension activates and creates a language client configured for JavaScript and TypeScript files [@test](../test/activation.test.ts)
- The language client is configured to use inter-process communication with the server [@test](../test/ipc-transport.test.ts)

### Custom analysis requests

The extension defines and sends custom requests to retrieve code suggestions from the server.

- A custom request type 'code/getSuggestions' is defined with position and URI parameters [@test](../test/custom-request-type.test.ts)
- The client can send the custom request and receive a response with suggestion text and ranges [@test](../test/send-custom-request.test.ts)

### Hover provider integration

When users hover over code, the extension retrieves and displays analysis from the server.

- A hover provider is registered for JavaScript and TypeScript files [@test](../test/hover-provider.test.ts)
- When hover is triggered, the extension sends a request to the server and displays the returned suggestion [@test](../test/hover-display.test.ts)

### Client lifecycle management

The extension manages the language client lifecycle appropriately.

- The client starts automatically when the extension activates [@test](../test/client-start.test.ts)
- The client stops gracefully when the extension is deactivated [@test](../test/client-stop.test.ts)

## Implementation

[@generates](./src/extension.ts)

## API

```typescript { #api }
/**
 * Activates the extension and initializes the language client
 */
export function activate(context: vscode.ExtensionContext): void;

/**
 * Deactivates the extension and stops the language client
 */
export function deactivate(): Thenable<void> | undefined;

/**
 * Parameters for code analysis request
 */
interface CodeAnalysisParams {
  uri: string;
  position: { line: number; character: number };
}

/**
 * Response from code analysis request
 */
interface CodeAnalysisResponse {
  suggestion: string;
  range: {
    start: { line: number; character: number };
    end: { line: number; character: number };
  };
}
```

## Dependencies { .dependencies }

### vscode-languageclient { .dependency }

Provides the language server protocol client for communicating with language servers.

[@satisfied-by](vscode-languageclient)
