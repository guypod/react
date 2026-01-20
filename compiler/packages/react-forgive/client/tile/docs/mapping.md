# Position Mapping

The position mapping module provides utilities for converting between Language Server Protocol (LSP) position formats and VS Code position objects. These conversions are essential for LSP client communication, enabling the extension to translate between the language server's coordinate system and VS Code's API.

## Capabilities

### Convert LSP Position to VS Code Position

Converts an LSP position literal (plain object with `line` and `character` properties) into a VS Code `Position` object.

```typescript { .api }
/**
 * Converts an LSP Position literal to a VS Code Position object
 * @param position - LSP position with line and character properties
 * @returns VS Code Position instance
 */
function positionLiteralToVSCodePosition(
  position: Position
): vscode.Position;
```

**Usage Example:**

```typescript
import { positionLiteralToVSCodePosition } from 'react-forgive-client/mapping';
import { Position } from 'vscode-languageclient/node';

// LSP position from language server
const lspPosition: Position = {
  line: 10,
  character: 5,
};

// Convert to VS Code position
const vscodePosition = positionLiteralToVSCodePosition(lspPosition);

// Use with VS Code APIs
vscode.window.activeTextEditor?.selection = new vscode.Selection(
  vscodePosition,
  vscodePosition
);
```

### Convert LSP Positions to VS Code Range

Converts two LSP position literals into a VS Code `Range` object, which represents a span of text in a document.

```typescript { .api }
/**
 * Converts two LSP Position literals into a VS Code Range object
 * @param start - Start position of the range
 * @param end - End position of the range
 * @returns VS Code Range spanning from start to end
 */
function positionsToRange(
  start: Position,
  end: Position
): vscode.Range;
```

**Usage Example:**

```typescript
import { positionsToRange } from 'react-forgive-client/mapping';
import { Position } from 'vscode-languageclient/node';

// LSP positions from language server
const startPos: Position = { line: 10, character: 5 };
const endPos: Position = { line: 10, character: 20 };

// Convert to VS Code range
const vscodeRange = positionsToRange(startPos, endPos);

// Use with VS Code APIs
vscode.window.activeTextEditor?.setDecorations(decorationType, [
  { range: vscodeRange },
]);

// Or for text replacement
vscode.window.activeTextEditor?.edit((editBuilder) => {
  editBuilder.replace(vscodeRange, 'new text');
});
```

## Position Format

### LSP Position Format

LSP uses plain objects with zero-based line and character indices:

```typescript
interface Position {
  /** Zero-based line number */
  line: number;
  /** Zero-based character offset within the line */
  character: number;
}
```

### VS Code Position Format

VS Code uses `Position` class instances with the same zero-based coordinate system:

```typescript
class Position {
  constructor(line: number, character: number);
  readonly line: number;
  readonly character: number;
}
```

Both formats use zero-based indexing, so the conversion is straightforward. The difference is that LSP typically sends plain objects (position literals) while VS Code APIs require `Position` class instances.

## VS Code Range Format

A VS Code `Range` represents a text span and is created from two `Position` instances:

```typescript
class Range {
  constructor(start: Position, end: Position);
  readonly start: Position;
  readonly end: Position;
}
```

The `positionsToRange()` function handles the conversion from LSP position pairs to VS Code ranges in a single step.

## Common Use Cases

### Handling LSP Responses

When the language server returns position data, convert it for use with VS Code APIs:

```typescript
import {
  positionLiteralToVSCodePosition,
  positionsToRange,
} from 'react-forgive-client/mapping';

// Language server response
interface ServerResponse {
  location: { start: Position; end: Position };
  relatedPositions: Position[];
}

function handleServerResponse(response: ServerResponse) {
  // Convert range
  const range = positionsToRange(
    response.location.start,
    response.location.end
  );

  // Convert individual positions
  const vscodePositions = response.relatedPositions.map(
    positionLiteralToVSCodePosition
  );

  // Use with VS Code APIs
  vscode.window.activeTextEditor?.revealRange(
    range,
    vscode.TextEditorRevealType.InCenter
  );
}
```

### Decoration Positioning

Convert LSP position data to apply decorations in the editor:

```typescript
import { positionsToRange } from 'react-forgive-client/mapping';
import { Position } from 'vscode-languageclient/node';

// Decorations from language server
const decorationData: Array<[Position, Position]> = [
  [
    { line: 5, character: 2 },
    { line: 5, character: 10 },
  ],
  [
    { line: 7, character: 2 },
    { line: 7, character: 15 },
  ],
];

// Convert and apply
const decorationOptions = decorationData.map(([start, end]) => ({
  range: positionsToRange(start, end),
  hoverMessage: 'Inferred dependency',
}));

vscode.window.activeTextEditor?.setDecorations(
  decorationType,
  decorationOptions
);
```

### Sending Positions to Language Server

When sending requests to the language server, you may need to convert VS Code positions to LSP format:

```typescript
import * as vscode from 'vscode';
import { Position } from 'vscode-languageclient/node';

// VS Code position
const vscodePosition = vscode.window.activeTextEditor!.selection.active;

// Convert to LSP format (plain object)
const lspPosition: Position = {
  line: vscodePosition.line,
  character: vscodePosition.character,
};

// Send to language server
client.sendRequest('custom/request', { position: lspPosition });
```

## Integration with Auto-Dependency System

The position mapping utilities are heavily used by the auto-dependency decoration system:

1. **Receiving Decorations**: Convert LSP position pairs from the server into VS Code ranges for decoration
2. **Tracking Locations**: Convert `useEffectCallExpr` position pairs to track currently decorated function locations
3. **User Interactions**: Convert VS Code positions from hover events to LSP format for server requests

This integration enables seamless communication between the VS Code UI and the language server analysis engine.
