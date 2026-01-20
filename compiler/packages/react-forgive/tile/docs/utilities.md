# Utilities

Helper functions for position mapping, range manipulation, and color utilities used throughout the React Forgive extension.

## Position Mapping Utilities

### Position Literal to VS Code Position

Converts LSP Position literal to VS Code Position object.

```typescript { .api }
/**
 * Converts LSP Position tuple to VS Code Position
 * @param position - LSP Position tuple [line, character]
 * @returns VS Code Position object
 */
function positionLiteralToVSCodePosition(position: Position): vscode.Position;

type Position = {
  line: number;
  character: number;
};
```

**Usage Example:**

```typescript
import { positionLiteralToVSCodePosition } from './mapping';

const lspPosition = { line: 10, character: 5 };
const vscodePosition = positionLiteralToVSCodePosition(lspPosition);
// Result: vscode.Position instance with line=10, character=5
```

### Positions to Range

Converts two LSP Positions to a VS Code Range.

```typescript { .api }
/**
 * Converts two LSP Positions to a VS Code Range
 * @param start - Start position
 * @param end - End position
 * @returns VS Code Range object
 */
function positionsToRange(start: Position, end: Position): vscode.Range;
```

**Usage Example:**

```typescript
import { positionsToRange } from './mapping';

const start = { line: 5, character: 0 };
const end = { line: 8, character: 10 };
const range = positionsToRange(start, end);
// Result: vscode.Range instance from (5,0) to (8,10)
```

## Range Utilities

### Range Type

Type definition for position ranges used in the server.

```typescript { .api }
/**
 * Tuple type representing a range with start and end positions
 */
type Range = [Position, Position];

type Position = {
  line: number;
  character: number;
};
```

### Is Position Within Range

Checks if a position's line falls within a range's line span.

```typescript { .api }
/**
 * Checks if a position is within a range
 * @param position - Position to check
 * @param range - Range tuple [start, end]
 * @returns true if position's line is within the range's line span
 */
function isPositionWithinRange(
  position: Position,
  [start, end]: Range,
): boolean;
```

**What it does:**

1. Checks if `position.line` is >= `start.line`
2. Checks if `position.line` is <= `end.line`
3. Returns true if both conditions are met

**Usage Example:**

```typescript
import { isPositionWithinRange } from './utils/range';

const position = { line: 10, character: 5 };
const range = [
  { line: 8, character: 0 },
  { line: 12, character: 0 }
];

const isWithin = isPositionWithinRange(position, range);
// Result: true (line 10 is between lines 8 and 12)
```

### Is Range Within Range

Checks if one range is completely contained within another range.

```typescript { .api }
/**
 * Checks if aRange is completely contained within bRange
 * @param aRange - Range to check
 * @param bRange - Container range
 * @returns true if aRange is fully within bRange (inclusive boundaries)
 */
function isRangeWithinRange(aRange: Range, bRange: Range): boolean;
```

**What it does:**

1. Checks if `aRange.start` is >= `bRange.start`
2. Checks if `aRange.end` is <= `bRange.end`
3. Uses inclusive boundary comparison
4. Returns true if aRange is completely contained

**Usage Example:**

```typescript
import { isRangeWithinRange } from './utils/range';

const innerRange = [
  { line: 10, character: 5 },
  { line: 12, character: 10 }
];

const outerRange = [
  { line: 8, character: 0 },
  { line: 15, character: 0 }
];

const isWithin = isRangeWithinRange(innerRange, outerRange);
// Result: true (innerRange is fully within outerRange)
```

### Source Location to Range

Converts Babel SourceLocation to LSP range tuple.

```typescript { .api }
/**
 * Converts Babel SourceLocation to LSP range tuple
 * @param loc - Babel SourceLocation with line and column info
 * @returns Range tuple [start Position, end Position] with adjusted line numbers
 */
function sourceLocationToRange(
  loc: t.SourceLocation,
): [Position, Position];
```

**What it does:**

1. Takes Babel SourceLocation (1-indexed lines)
2. Adjusts line numbers to 0-indexed for LSP
3. Returns range tuple with start and end positions

**Usage Example:**

```typescript
import { sourceLocationToRange } from './utils/range';
import * as t from '@babel/types';

const babelLoc: t.SourceLocation = {
  start: { line: 10, column: 4 },
  end: { line: 10, column: 20 }
};

const range = sourceLocationToRange(babelLoc);
// Result: [{ line: 9, character: 4 }, { line: 9, character: 20 }]
```

## Color Utilities

### Color Class

Utility class for color manipulation and CSS string generation.

```typescript { .api }
/**
 * Color utility class for RGB color manipulation
 */
class Color {
  /**
   * Creates a new Color instance
   * @param r - Red component (0-255)
   * @param g - Green component (0-255)
   * @param b - Blue component (0-255)
   */
  constructor(r: number, g: number, b: number);

  /**
   * Returns CSS rgba string with specified transparency
   * @param a - Alpha transparency (0-1)
   * @returns CSS rgba string (e.g., "rgba(255, 0, 0, 0.5)")
   */
  toAlphaString(a: number): string;

  /**
   * Returns CSS rgba string with full opacity
   * @returns CSS rgba string with alpha=1 (e.g., "rgba(255, 0, 0, 1)")
   */
  toString(): string;

  /**
   * Returns a new Color with RGB values multiplied by a factor
   * @param mult - Multiplication factor for RGB values
   * @returns New Color instance with adjusted values
   */
  adjusted(mult: number): Color;
}
```

**Usage Examples:**

```typescript
import { Color } from './colors';

// Create a red color
const red = new Color(255, 0, 0);

// Get CSS string with 50% transparency
const transparentRed = red.toAlphaString(0.5);
// Result: "rgba(255, 0, 0, 0.5)"

// Get CSS string with full opacity
const opaqueRed = red.toString();
// Result: "rgba(255, 0, 0, 1)"

// Create a darker shade (50% brightness)
const darkRed = red.adjusted(0.5);
// Result: Color(127, 0, 0)
```

### Color Constants

Predefined color constants for convenience.

```typescript { .api }
/**
 * Black color constant
 */
const BLACK: Color;

/**
 * White color constant
 */
const WHITE: Color;
```

**Usage Example:**

```typescript
import { BLACK, WHITE } from './colors';

console.log(BLACK.toString());  // "rgba(0, 0, 0, 1)"
console.log(WHITE.toString());  // "rgba(255, 255, 255, 1)"
```

### Get Color For Index

Returns a color from a predefined pool of 10 distinct colors.

```typescript { .api }
/**
 * Gets a color from a predefined pool based on index
 * @param index - Index to select color (uses modulo 10 to cycle through colors)
 * @returns Color instance from the predefined pool
 */
function getColorFor(index: number): Color;
```

**What it does:**

1. Maintains a pool of 10 distinct colors
2. Uses `index % 10` to cycle through the color pool
3. Returns a Color instance for visualization

**Color Pool:**
The function cycles through 10 predefined colors suitable for syntax highlighting and decorations.

**Usage Example:**

```typescript
import { getColorFor } from './colors';

// Get colors for multiple items
const color0 = getColorFor(0);  // First color
const color1 = getColorFor(1);  // Second color
const color10 = getColorFor(10); // Same as color0 (cycles through)

// Use in decoration
const decorationType = vscode.window.createTextEditorDecorationType({
  borderColor: getColorFor(index).toAlphaString(0.7)
});
```

## Module Locations

**Files**:
- `client/src/mapping.ts` - Position and range conversion for client
- `client/src/colors.ts` - Color utilities for visualizations
- `server/src/utils/range.ts` - Range utilities for server

## Related APIs

- [Auto-Dependency Decorations](./auto-deps.md) - Uses position/range utilities for decorations
- [Compilation](./compilation.md) - Uses sourceLocationToRange for Babel conversions
- [LSP Server](./lsp-server.md) - Uses range utilities in event handlers
