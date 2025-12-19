# React Forgive Client

React Forgive Client is an experimental Visual Studio Code extension that provides a Language Server Protocol (LSP) client for React development. It offers automatic dependency analysis and visualization for React components, particularly focusing on React hooks like `useEffect`. The extension provides real-time visual feedback about component dependencies through code decorations and hover interactions.

## Package Information

- **Package Name**: react-forgive-client
- **Package Type**: npm (VS Code Extension)
- **Language**: TypeScript
- **Installation**: Part of React compiler toolchain (not published to npm)
- **Target Languages**: JavaScript React (JSX), TypeScript React (TSX)

## Core Imports

This package is a VS Code extension, so imports are used internally within the extension code rather than by external consumers. The main exports are for VS Code extension activation:

```typescript
import { activate, deactivate } from 'react-forgive-client';
```

Internal module imports for extension development:

```typescript
import {
  requestAutoDepsDecorations,
  getCurrentlyDecoratedAutoDepFnLoc,
  type AutoDepsDecorationsLSPEvent,
  type AutoDepsDecorationsParams,
} from 'react-forgive-client/autodeps';

import { Color, BLACK, WHITE, getColorFor } from 'react-forgive-client/colors';

import {
  positionLiteralToVSCodePosition,
  positionsToRange,
} from 'react-forgive-client/mapping';
```

## Basic Usage

As a VS Code extension, this package is used by activating it in VS Code. The extension automatically:

1. Starts an LSP client that connects to the React Forgive language server
2. Registers hover providers for JSX/TSX files
3. Listens for document changes to update decorations
4. Provides a command `react.requestAutoDepsDecorations` for manual decoration requests

The extension visualizes inferred React hook dependencies by underlining them in the editor when hovering over or working with `useEffect` calls.

## Architecture

The extension is built around several key components:

- **Extension Activation**: Main entry point (`activate`/`deactivate`) that initializes the LSP client and registers VS Code providers
- **LSP Client Integration**: Communicates with a language server to analyze React code patterns
- **Auto-Dependency System**: Analyzes React hooks and visualizes inferred dependencies through editor decorations
- **Color Utilities**: Provides color management for visual decorations with RGB manipulation
- **Position Mapping**: Converts between LSP position formats and VS Code position objects

## Capabilities

### Extension Lifecycle

Core VS Code extension lifecycle functions for initialization and cleanup.

```typescript { .api }
/**
 * Activates the VS Code extension, initializing the LSP client and registering providers
 * @param context - VS Code extension context
 */
function activate(context: vscode.ExtensionContext): void;

/**
 * Deactivates the extension and cleans up resources
 * @returns Promise that resolves when cleanup is complete, or undefined if no cleanup needed
 */
function deactivate(): Thenable<void> | undefined;
```

### Auto-Dependency Decorations

Automatic dependency analysis and visualization for React hooks, particularly `useEffect`. Provides LSP-based inference of hook dependencies and renders visual decorations in the editor.

```typescript { .api }
/**
 * Requests auto-dependency decorations from the language server
 * @param client - The LSP client instance
 * @param position - VS Code position to analyze
 * @param options - Configuration for decoration behavior
 */
function requestAutoDepsDecorations(
  client: LanguageClient,
  position: vscode.Position,
  options: AutoDepsDecorationsOptions
): void;

interface AutoDepsDecorationsOptions {
  /** Whether to update the currently tracked decoration location */
  shouldUpdateCurrent: boolean;
}

interface AutoDepsDecorationsLSPEvent {
  /** Start and end positions of the useEffect call expression */
  useEffectCallExpr: [Position, Position];
  /** Array of position pairs defining where decorations should be applied */
  decorations: Array<[Position, Position]>;
}

interface AutoDepsDecorationsParams {
  /** Position in the document to request decorations for */
  position: Position;
}
```

[Auto-Dependency Decorations](./autodeps.md)

### Color Utilities

RGB color management with support for lightening/darkening and CSS string conversion. Used for creating visual decorations in the editor.

```typescript { .api }
class Color {
  constructor(r: number, g: number, b: number);
  /** Convert to CSS rgba string with specified alpha value */
  toAlphaString(a: number): string;
  /** Convert to CSS rgba string with alpha=1 */
  toString(): string;
  /** Create adjusted color by multiplier (>1 lightens, <1 darkens) */
  adjusted(mult: number): Color;
}

/** Pre-defined black color (0, 0, 0) */
const BLACK: Color;

/** Pre-defined white color (255, 255, 255) */
const WHITE: Color;

/**
 * Get a color from the predefined color pool
 * @param index - Index to select color (wraps using modulo)
 */
function getColorFor(index: number): Color;
```

[Color Utilities](./colors.md)

### Position Mapping

Utilities for converting between LSP position formats and VS Code position objects, essential for LSP client communication.

```typescript { .api }
/**
 * Convert LSP Position literal to VS Code Position object
 * @param position - LSP position with line and character properties
 */
function positionLiteralToVSCodePosition(
  position: Position
): vscode.Position;

/**
 * Convert two LSP positions into a VS Code Range
 * @param start - Start position
 * @param end - End position
 */
function positionsToRange(
  start: Position,
  end: Position
): vscode.Range;
```

[Position Mapping](./mapping.md)

## LSP Integration

The extension communicates with a React Forgive language server using a custom LSP request:

- **Request Type**: `react/autodeps_decorations`
- **Transport**: IPC (Inter-Process Communication)
- **Server Module**: `dist/server.js` (relative to extension root)

## Registered Commands

- `react.requestAutoDepsDecorations` - Manually request auto-dependency decorations for a specific position

## File Support

- `javascriptreact` (JSX files)
- `typescriptreact` (TSX files)
- File scheme: `file://`
