# LSP Decoration Utilities

A TypeScript utility module for VS Code extensions that work with Language Server Protocol (LSP) data. The module provides functions to convert LSP position data, manage visual decorations, manipulate colors with hue preservation, and handle async decoration requests with race condition prevention.

## Capabilities

### Position Conversion

- It converts LSP position literals to VS Code Position objects [@test](../test/position-conversion.test.ts)
- It creates VS Code Range objects from pairs of LSP positions [@test](../test/range-creation.test.ts)

### Decoration Management

- It applies text decorations with hover messages to the active editor [@test](../test/apply-decorations.test.ts)
- It clears all decorations when given an empty array [@test](../test/clear-decorations.test.ts)

### Color Manipulation

- It adjusts colors while preserving hue using the redistribution algorithm [@test](../test/color-adjust.test.ts)
- It formats colors as CSS RGBA strings with alpha transparency [@test](../test/color-format.test.ts)

### Async Request Handling

- It processes decoration requests with race condition prevention [@test](../test/request-handling.test.ts)
- It discards stale responses when newer requests are made [@test](../test/stale-response.test.ts)

## Implementation

[@generates](./src/index.ts)

## API

```typescript { #api }
import * as vscode from 'vscode';
import { Position } from 'vscode-languageclient/node';

/**
 * Converts an LSP Position literal to a VS Code Position object
 */
export function positionLiteralToVSCodePosition(position: Position): vscode.Position;

/**
 * Creates a VS Code Range from start and end LSP positions
 */
export function positionsToRange(start: Position, end: Position): vscode.Range;

/**
 * Applies decorations to the active text editor with the given ranges and hover messages
 */
export function drawDecorations(
  decorationType: vscode.TextEditorDecorationType,
  decorations: Array<{ start: Position; end: Position; message: string }>
): void;

/**
 * Clears all decorations of the specified type from the active editor
 */
export function clearDecorations(decorationType: vscode.TextEditorDecorationType): void;

/**
 * Color class with adjustment capabilities
 */
export class Color {
  constructor(r: number, g: number, b: number);

  /**
   * Returns CSS RGBA string with specified alpha value
   */
  toAlphaString(a: number): string;

  /**
   * Returns CSS RGB string
   */
  toString(): string;

  /**
   * Adjusts color brightness by multiplier while preserving hue
   * Values > 1.0 lighten, values < 1.0 darken
   */
  adjusted(mult: number): Color;
}

/**
 * Options for handling decoration requests
 */
export interface RequestOptions {
  /**
   * Whether to update the currently tracked decoration location
   */
  shouldUpdateLocation: boolean;
}

/**
 * Manages sequential decoration requests with race condition prevention.
 * Increments and returns a request ID, executes the request function,
 * and only applies decorations if the response matches the latest request ID.
 */
export function requestDecorations(
  decorationType: vscode.TextEditorDecorationType,
  requestFn: () => Promise<Array<{ start: Position; end: Position; message: string }> | null>
): void;
```

## Dependencies { .dependencies }

### react-forgive-client { .dependency }

Provides LSP client utilities for VS Code extensions, including position conversion, decoration management, and color manipulation with hue-preserving algorithms.
