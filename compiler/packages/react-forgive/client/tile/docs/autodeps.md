# Auto-Dependency Decorations

The auto-dependency decorations system provides automatic analysis and visualization of React hook dependencies. It communicates with a language server to identify inferred dependencies and displays them as visual decorations in the VS Code editor.

## Capabilities

### Request Auto-Dependency Decorations

Sends a request to the language server to analyze a position in the document and retrieve inferred dependency information for React hooks.

```typescript { .api }
/**
 * Requests auto-dependency decorations from the language server and applies them to the editor
 * @param client - The Language Client instance to send the request through
 * @param position - The position in the document to analyze
 * @param options - Options controlling decoration behavior
 */
function requestAutoDepsDecorations(
  client: LanguageClient,
  position: vscode.Position,
  options: AutoDepsDecorationsOptions
): void;

interface AutoDepsDecorationsOptions {
  /**
   * Whether to update the currently tracked decoration location
   * Set to true when initiating a new decoration request
   * Set to false when refreshing existing decorations
   */
  shouldUpdateCurrent: boolean;
}
```

**Usage Example:**

```typescript
import { requestAutoDepsDecorations } from 'react-forgive-client/autodeps';
import { LanguageClient } from 'vscode-languageclient/node';
import * as vscode from 'vscode';

// Inside extension activation
const client = new LanguageClient(/* ... */);
const position = new vscode.Position(10, 5);

// Request decorations for a specific position
requestAutoDepsDecorations(client, position, {
  shouldUpdateCurrent: true,
});
```

### Get Currently Decorated Function Location

Retrieves the range of the currently decorated auto-dependency function (e.g., the `useEffect` call).

```typescript { .api }
/**
 * Gets the currently decorated auto-dependency function location
 * @returns The VS Code Range of the current decoration, or null if none exists
 */
function getCurrentlyDecoratedAutoDepFnLoc(): vscode.Range | null;
```

### Set Currently Decorated Function Location

Updates the tracked location of the currently decorated function.

```typescript { .api }
/**
 * Sets the currently decorated auto-dependency function location
 * @param range - The VS Code Range to set as the current decoration location
 */
function setCurrentlyDecoratedAutoDepFnLoc(range: vscode.Range): void;
```

### Clear Currently Decorated Function Location

Clears the tracked location of the currently decorated function.

```typescript { .api }
/**
 * Clears the currently decorated auto-dependency function location
 * Sets the internal tracking to null
 */
function clearCurrentlyDecoratedAutoDepFnLoc(): void;
```

### Draw Inferred Effect Dependency Decorations

Renders visual decorations in the editor for inferred effect dependencies.

```typescript { .api }
/**
 * Draws inferred effect dependency decorations in the active text editor
 * @param decorations - Array of position pairs defining decoration ranges
 */
function drawInferredEffectDepDecorations(
  decorations: Array<[Position, Position]>
): void;
```

**Usage Example:**

```typescript
import { drawInferredEffectDepDecorations } from 'react-forgive-client/autodeps';
import { Position } from 'vscode-languageclient/node';

// Draw decorations for inferred dependencies
const decorations: Array<[Position, Position]> = [
  [
    { line: 5, character: 2 },
    { line: 5, character: 10 },
  ],
  [
    { line: 7, character: 2 },
    { line: 7, character: 15 },
  ],
];

drawInferredEffectDepDecorations(decorations);
```

### Clear Decorations

Removes all decorations of a specified type from the active text editor.

```typescript { .api }
/**
 * Clears all decorations of the specified type from the active text editor
 * @param decorationType - The VS Code TextEditorDecorationType to clear
 */
function clearDecorations(
  decorationType: vscode.TextEditorDecorationType
): void;
```

## LSP Types

### Auto-Dependency Decorations Request

Defines the LSP request type for auto-dependency decorations.

```typescript { .api }
namespace AutoDepsDecorationsRequest {
  /**
   * LSP request type for requesting auto-dependency decorations
   * Request identifier: 'react/autodeps_decorations'
   */
  export const type: RequestType<
    AutoDepsDecorationsParams,
    AutoDepsDecorationsLSPEvent | null,
    void
  >;
}
```

### LSP Event Structure

The response from the language server when auto-dependency decorations are requested.

```typescript { .api }
interface AutoDepsDecorationsLSPEvent {
  /**
   * Start and end positions of the useEffect call expression
   * Tuple format: [startPosition, endPosition]
   */
  useEffectCallExpr: [Position, Position];
  /**
   * Array of position pairs defining where decorations should be applied
   * Each tuple represents a range to be decorated
   */
  decorations: Array<[Position, Position]>;
}
```

### LSP Request Parameters

Parameters sent to the language server when requesting decorations.

```typescript { .api }
interface AutoDepsDecorationsParams {
  /**
   * The position in the document to analyze for auto-dependency decorations
   */
  position: Position;
}
```

## Decoration Styling

Decorations use the following default styling:

- **Border Color**: VS Code theme color `diffEditor.move.border`
- **Border Style**: Solid line
- **Border Width**: 4px bottom border (underline effect)
- **Hover Message**: "Inferred as an effect dependency"

The styling creates an underline effect beneath inferred dependencies to make them visually distinct without being intrusive.

## Request Sequencing

The decoration system uses request IDs to maintain proper ordering and prevent race conditions. When a new decoration request is sent before a previous one completes, only the most recent response is applied to the editor.

## Integration with VS Code

The auto-dependency decoration system integrates with VS Code through:

1. **Hover Provider**: Triggers decoration requests when users hover over code
2. **Document Change Listener**: Updates decorations when the document is edited
3. **Command**: Exposes `react.requestAutoDepsDecorations` command for manual invocation

All decorations are applied to the active text editor and automatically cleared when no longer relevant.
