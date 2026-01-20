# Color Utilities

The color utilities module provides RGB color management with support for color manipulation and CSS string conversion. These utilities are used for creating and styling visual decorations in the VS Code editor.

## Capabilities

### Color Class

The `Color` class represents an RGB color and provides methods for manipulation and conversion.

```typescript { .api }
/**
 * Represents an RGB color with manipulation capabilities
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
   * Converts the color to a CSS rgba string with specified alpha
   * @param a - Alpha value (0-1)
   * @returns CSS rgba string, e.g., "rgba(255,0,0,0.5)"
   */
  toAlphaString(a: number): string;

  /**
   * Converts the color to a CSS rgba string with full opacity
   * @returns CSS rgba string with alpha=1, e.g., "rgba(255,0,0,1)"
   */
  toString(): string;

  /**
   * Creates a new Color instance adjusted by a multiplier
   * Multiplier > 1.0 lightens the color
   * Multiplier < 1.0 darkens the color
   * Maintains hue while adjusting brightness
   * @param mult - Adjustment multiplier
   * @returns New Color instance with adjusted values
   */
  adjusted(mult: number): Color;
}
```

**Usage Examples:**

```typescript
import { Color } from 'react-forgive-client/colors';

// Create a red color
const red = new Color(255, 0, 0);

// Convert to CSS string
console.log(red.toString()); // "rgba(255,0,0,1)"
console.log(red.toAlphaString(0.5)); // "rgba(255,0,0,0.5)"

// Lighten the color
const lightRed = red.adjusted(1.5);
console.log(lightRed.toString()); // Lighter red

// Darken the color
const darkRed = red.adjusted(0.5);
console.log(darkRed.toString()); // Darker red

// Create custom colors
const purple = new Color(128, 0, 128);
const orange = new Color(255, 165, 0);
```

### Pre-defined Colors

The module exports two standard color constants.

```typescript { .api }
/**
 * Pre-defined black color
 * RGB values: (0, 0, 0)
 */
const BLACK: Color;

/**
 * Pre-defined white color
 * RGB values: (255, 255, 255)
 */
const WHITE: Color;
```

**Usage Example:**

```typescript
import { BLACK, WHITE } from 'react-forgive-client/colors';

// Use pre-defined colors
console.log(BLACK.toString()); // "rgba(0,0,0,1)"
console.log(WHITE.toString()); // "rgba(255,255,255,1)"

// Create variations
const gray = BLACK.adjusted(0.5);
const lightGray = WHITE.adjusted(0.8);
```

### Color Pool Selection

Retrieves colors from a predefined pool of visually distinct colors, useful for assigning colors to multiple items.

```typescript { .api }
/**
 * Gets a color from the predefined color pool based on an index
 * The pool contains 10 visually distinct colors
 * Index wraps around using modulo, so any integer is valid
 * @param index - Index to select from the color pool
 * @returns Color instance from the predefined pool
 */
function getColorFor(index: number): Color;
```

**Usage Example:**

```typescript
import { getColorFor } from 'react-forgive-client/colors';

// Get colors for multiple items
const items = ['item1', 'item2', 'item3'];
items.forEach((item, index) => {
  const color = getColorFor(index);
  console.log(`${item}: ${color.toString()}`);
});

// Index wraps around, so these give the same color
const color0 = getColorFor(0);
const color10 = getColorFor(10);
const color20 = getColorFor(20);
// All three are the same color

// Negative indices also work
const colorNeg5 = getColorFor(-5);
```

## Color Pool

The predefined color pool contains 10 carefully selected colors that are visually distinct and suitable for editor decorations:

1. Red: RGB(249, 65, 68)
2. Orange-Red: RGB(243, 114, 44)
3. Orange: RGB(248, 150, 30)
4. Light Orange: RGB(249, 132, 74)
5. Yellow: RGB(249, 199, 79)
6. Green: RGB(144, 190, 109)
7. Teal: RGB(67, 170, 139)
8. Blue-Green: RGB(77, 144, 142)
9. Blue: RGB(87, 117, 144)
10. Dark Blue: RGB(39, 125, 161)

## Color Adjustment Algorithm

The `adjusted()` method uses a color redistribution algorithm that maintains hue while adjusting brightness. The algorithm:

1. Multiplies each RGB component by the adjustment factor
2. If any component exceeds 255, redistributes the excess while maintaining ratios
3. Clamps values to the valid RGB range (0-255)
4. Returns integer values for each component

This approach ensures that colors remain visually consistent when lightened or darkened, unlike simple clamping which can shift hues.

## Integration with VS Code

Colors are primarily used to style text editor decorations. The `toAlphaString()` and `toString()` methods produce CSS rgba strings that are compatible with VS Code's decoration API:

```typescript
import { Color } from 'react-forgive-client/colors';
import * as vscode from 'vscode';

const decorationColor = new Color(249, 65, 68);

const decorationType = vscode.window.createTextEditorDecorationType({
  borderColor: decorationColor.toAlphaString(0.8),
  backgroundColor: decorationColor.toAlphaString(0.1),
});
```
