# React Hook Visualizer

A VS Code extension client component that displays visual decorations for React hooks using custom Language Server Protocol requests.

## Capabilities

### Custom LSP Request

Defines and sends a custom LSP request to retrieve decoration information for React hooks at a document position.

- A custom request type is defined with the identifier `'react/hookDecorations'` that accepts position parameters and returns decoration ranges or null [@test](../test/custom-request-type.test.ts)
- When requesting decorations at a position, the client sends the request with `HookDecorationsParams` containing the position [@test](../test/send-request.test.ts)
- The request returns `HookDecorationsResponse` with `hookCallExpr` range and `decorations` array when a hook is found at the position [@test](../test/request-response.test.ts)

### Text Editor Decorations

Applies visual decorations with styled borders and hover messages to ranges in the active editor.

- A decoration type is created with a bottom border using the theme color `'diffEditor.move.border'` [@test](../test/decoration-style.test.ts)
- Decorations are applied to the active editor with ranges and hover messages saying "Hook dependency" [@test](../test/apply-decorations.test.ts)

### Position Conversion Utilities

Converts between LSP position/range formats and VS Code position/range objects.

- A position literal object `{line, character}` is converted to a `vscode.Position` object [@test](../test/position-conversion.test.ts)
- Two position literals are converted to a `vscode.Range` object with start and end positions [@test](../test/range-conversion.test.ts)

## Implementation

[@generates](./src/extension.ts)

## API

```typescript { #api }
import * as vscode from 'vscode';
import { LanguageClient, RequestType } from 'vscode-languageclient/node';

/**
 * Position in a text document expressed as zero-based line and character offset
 */
interface Position {
  line: number;
  character: number;
}

/**
 * Range in a text document represented as start and end positions
 */
type Range = [Position, Position];

/**
 * Parameters for requesting hook decorations at a specific position
 */
interface HookDecorationsParams {
  position: Position;
}

/**
 * Response containing hook location and decoration ranges
 */
interface HookDecorationsResponse {
  hookCallExpr: Range;
  decorations: Array<Range>;
}

/**
 * Custom LSP request type namespace for hook decorations
 */
namespace HookDecorationsRequest {
  export const type: RequestType<
    HookDecorationsParams,
    HookDecorationsResponse | null,
    void
  >;
}

/**
 * Sends a custom LSP request for hook decorations at the given position
 *
 * @param client - The LSP language client
 * @param position - The position in the document to query
 * @returns Promise resolving to decoration information or null
 */
export function requestHookDecorations(
  client: LanguageClient,
  position: vscode.Position
): Promise<HookDecorationsResponse | null>;

/**
 * Applies visual decorations to the active editor for the given ranges
 *
 * @param decorations - Array of position ranges to decorate
 */
export function drawDecorations(
  decorations: Array<Range>
): void;

/**
 * Converts a Position literal to a VS Code Position object
 *
 * @param position - Position literal with line and character
 * @returns VS Code Position object
 */
export function convertPosition(
  position: Position
): vscode.Position;

/**
 * Converts two positions to a VS Code Range object
 *
 * @param start - Start position
 * @param end - End position
 * @returns VS Code Range object
 */
export function convertRange(
  start: Position,
  end: Position
): vscode.Range;
```

## Dependencies { .dependencies }

### vscode { .dependency }

Provides VS Code extension APIs including TextEditorDecorationType, Position, Range, and window management.

[@satisfied-by](vscode)

### vscode-languageclient { .dependency }

Provides Language Server Protocol client APIs including LanguageClient and RequestType for custom LSP requests.

[@satisfied-by](vscode-languageclient)
