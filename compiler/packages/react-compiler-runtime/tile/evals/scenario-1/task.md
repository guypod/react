# React Performance Debugger

Build a development tool that helps React developers debug performance issues by tracking component renders and detecting unintended value changes in memoization dependencies.

## Capabilities

### Render count tracking

- Calling `trackComponentRender("UserProfile")` for the first time returns 0, and a second call returns 1 [@test](../test/render-tracking.test.ts)
- Calling `trackComponentRender("Header")` twice and `trackComponentRender("Footer")` once tracks them independently [@test](../test/multiple-components.test.ts)
- After tracking renders for multiple components, calling `clearAllCounters()` resets all counts, then calling `trackComponentRender("Header")` returns 0 [@test](../test/clear-counters.test.ts)

### Value change detection

- Calling `detectValueChanges({a: 1}, {a: 1, b: 2}, "props", "MyComponent")` logs an error to console showing that property "b" was added [@test](../test/object-property-added.test.ts)
- Calling `detectValueChanges([1, 2], [1, 2, 3], "items", "ListComponent")` logs an error showing the array length changed from 2 to 3 [@test](../test/array-length-change.test.ts)
- Calling `detectValueChanges(mapWithOneEntry, mapWithTwoEntries, "cache", "DataFetcher")` logs an error showing the Map size changed from 1 to 2 [@test](../test/map-size-change.test.ts)

### Performance report generation

- After calling `trackComponentRender("App")` twice and `trackComponentRender("Sidebar")` once, `generatePerformanceReport()` returns `{"App": 2, "Sidebar": 1}` [@test](../test/generate-report.test.ts)

## Implementation

[@generates](../src/debugger.ts)

## API

```typescript { #api }
/**
 * Registers a component for render tracking and increments its count.
 * Returns the current render count for that component after incrementing.
 * On first call for a component, initializes to 0 and returns 0.
 */
export function trackComponentRender(componentName: string): number;

/**
 * Clears all render counters back to 0 for all tracked components.
 */
export function clearAllCounters(): void;

/**
 * Detects structural differences between two values and logs detailed
 * information about what changed. Logs include the variable name,
 * function name, and the specific path where changes occurred.
 */
export function detectValueChanges(
  oldValue: any,
  newValue: any,
  variableName: string,
  functionName: string
): void;

/**
 * Returns a performance report with all tracked components and their
 * render counts. Returns an object mapping component names to their
 * render counts.
 */
export function generatePerformanceReport(): Record<string, number>;
```

## Dependencies { .dependencies }

### react-compiler-runtime { .dependency }

Provides runtime instrumentation APIs for tracking renders and structural validation.

[@satisfied-by](react-compiler-runtime)
