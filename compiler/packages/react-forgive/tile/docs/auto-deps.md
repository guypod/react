# Auto-Dependency Decorations

Auto-dependency decoration functionality enables the visualization of inferred React hook dependencies in the VS Code editor. This is the core feature that shows developers which values should be included in their effect dependency arrays.

## Capabilities

### Request Auto-Deps Decorations

Requests auto-dependency decoration data from the LSP server and updates the editor UI.

```typescript { .api }
/**
 * Requests auto-dependency decorations from the LSP server
 * @param client - LanguageClient instance for sending LSP requests
 * @param position - VS Code position (cursor location) to check for effect dependencies
 * @param options - Options controlling decoration behavior
 */
function requestAutoDepsDecorations(
  client: LanguageClient,
  position: vscode.Position,
  options: AutoDepsDecorationsOptions,
): void;
```

**What it does:**

1. Sends a custom LSP request `react/autodeps_decorations` to the server
2. Receives decoration data for the position (if within a useEffect call)
3. Calls `drawInferredEffectDepDecorations()` to apply visual decorations
4. Clears decorations if response is null (position not in an eligible function)

**Usage Example:**

```typescript
import { requestAutoDepsDecorations } from './autodeps';
import { LanguageClient } from 'vscode-languageclient/node';

// Request decorations for current cursor position
requestAutoDepsDecorations(
  client,
  editor.selection.active,
  { shouldUpdateCurrent: true }
);
```

### Draw Inferred Effect Dep Decorations

Applies visual decorations to inferred effect dependencies in the active editor.

```typescript { .api }
/**
 * Applies visual decorations to inferred effect dependencies
 * @param decorations - Array of position ranges to decorate
 */
function drawInferredEffectDepDecorations(
  decorations: Array<[Position, Position]>,
): void;
```

**What it does:**

1. Converts LSP positions to VS Code ranges
2. Creates `TextEditorDecorationType` with bottom border styling
3. Applies decorations to the active editor
4. Sets hover message: "Inferred as an effect dependency"

**Decoration Styling:**

- **Border Color**: `diffEditor.move.border` (VS Code theme color)
- **Border Style**: Solid bottom border (4px width)
- **Hover Message**: "Inferred as an effect dependency"

**Usage Example:**

```typescript
import { drawInferredEffectDepDecorations } from './autodeps';

// Apply decorations for inferred dependencies
const decorations = [
  [{ line: 10, character: 4 }, { line: 10, character: 12 }],
  [{ line: 11, character: 4 }, { line: 11, character: 18 }]
];
drawInferredEffectDepDecorations(decorations);
```

### Clear Decorations

Removes all decorations of a given type from the active editor.

```typescript { .api }
/**
 * Removes all decorations from the active editor
 * @param decorationType - The decoration type to clear
 */
function clearDecorations(
  decorationType: vscode.TextEditorDecorationType,
): void;
```

**What it does:**

1. Gets the active text editor
2. Clears decorations by setting an empty range array
3. Does nothing if no active editor exists

**Usage Example:**

```typescript
import { clearDecorations } from './autodeps';
import * as vscode from 'vscode';

const decorationType = vscode.window.createTextEditorDecorationType({
  // decoration options
});

// Clear all decorations of this type
clearDecorations(decorationType);
```

### Get Currently Decorated Auto-Dep Function Location

Returns the VS Code range of the currently decorated effect dependency function.

```typescript { .api }
/**
 * Gets the currently decorated effect dependency function location
 * @returns VS Code range of the decorated function, or null if none
 */
function getCurrentlyDecoratedAutoDepFnLoc(): vscode.Range | null;
```

**Usage Example:**

```typescript
import { getCurrentlyDecoratedAutoDepFnLoc } from './autodeps';

const currentRange = getCurrentlyDecoratedAutoDepFnLoc();
if (currentRange) {
  console.log('Currently decorating:', currentRange);
}
```

### Set Currently Decorated Auto-Dep Function Location

Sets the VS Code range of the currently decorated effect dependency function.

```typescript { .api }
/**
 * Sets the currently decorated effect dependency function location
 * @param range - VS Code range to track as the current decoration
 */
function setCurrentlyDecoratedAutoDepFnLoc(range: vscode.Range): void;
```

**Usage Example:**

```typescript
import { setCurrentlyDecoratedAutoDepFnLoc } from './autodeps';
import * as vscode from 'vscode';

const range = new vscode.Range(10, 0, 15, 0);
setCurrentlyDecoratedAutoDepFnLoc(range);
```

### Clear Currently Decorated Auto-Dep Function Location

Clears the tracking state for the currently decorated effect dependency function.

```typescript { .api }
/**
 * Clears the currently decorated effect dependency function location
 */
function clearCurrentlyDecoratedAutoDepFnLoc(): void;
```

**Usage Example:**

```typescript
import { clearCurrentlyDecoratedAutoDepFnLoc } from './autodeps';

// Clear the current decoration tracking state
clearCurrentlyDecoratedAutoDepFnLoc();
```

## Types

### AutoDepsDecorationsLSPEvent

Response structure from the LSP server containing decoration data.

```typescript { .api }
/**
 * LSP event containing auto-dependency decoration data
 */
type AutoDepsDecorationsLSPEvent = {
  /** Position range of the useEffect call expression */
  useEffectCallExpr: [Position, Position];
  /** Array of position ranges for inferred dependencies */
  decorations: Array<[Position, Position]>;
};
```

### AutoDepsDecorationsParams

Request parameters for auto-dependency decorations.

```typescript { .api }
/**
 * Parameters for requesting auto-dependency decorations
 */
interface AutoDepsDecorationsParams {
  /** LSP position to check for effect dependencies */
  position: Position;
}
```

### AutoDepsDecorationsOptions

Options controlling decoration request behavior.

```typescript { .api }
/**
 * Options for requesting auto-dependency decorations
 */
type AutoDepsDecorationsOptions = {
  /** Whether to update the currently tracked decoration location */
  shouldUpdateCurrent: boolean;
};
```

### AutoDepsDecorationsRequest

Namespace containing the LSP request type definition.

```typescript { .api }
/**
 * LSP request type for auto-dependency decorations
 */
namespace AutoDepsDecorationsRequest {
  /**
   * Request type for 'react/autodeps_decorations' custom LSP request
   */
  const type: RequestType<
    AutoDepsDecorationsParams,
    AutoDepsDecorationsLSPEvent | null,
    void
  >;
}
```

**Request Details:**

- **Request Method**: `react/autodeps_decorations`
- **Request Params**: `{ position: Position }`
- **Response (Success)**: `{ useEffectCallExpr: [Position, Position], decorations: Array<[Position, Position]> }`
- **Response (No Match)**: `null`

## LSP Position Type

```typescript { .api }
/**
 * LSP position type (0-indexed)
 */
type Position = {
  line: number;
  character: number;
};
```

## Module Location

**File**: `client/src/autodeps.ts`

## Related APIs

- [Extension Lifecycle](./extension-lifecycle.md) - Registers hover providers that trigger decoration requests
- [LSP Server](./lsp-server.md) - Handles the `react/autodeps_decorations` request on the server side
- [Utilities](./utilities.md) - Position and range conversion helpers
