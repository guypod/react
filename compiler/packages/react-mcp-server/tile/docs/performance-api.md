# Performance API

Runtime performance measurement for React components using Web Vitals and React Profiler.

## Capabilities

### Measure Performance

Measures React component runtime performance by rendering in a headless browser and collecting metrics across multiple iterations.

```typescript { .api }
/**
 * Measure React component performance with Web Vitals and React Profiler
 * @param code - React component code to measure (must contain function App())
 * @param iterations - Number of measurement iterations
 * @returns Promise resolving to aggregated performance results
 */
function measurePerformance(
  code: string,
  iterations: number
): Promise<PerformanceResults>;
```

**Code Requirements**:
- Must contain `function App()` declaration (not arrow function)
- No export statements allowed
- Only import React from 'react'
- Use `React.` prefix for all hooks (e.g., `React.useState`, `React.useEffect`)

**Measurement Process**:

1. **Transpile**: Transform code with Babel
   - Presets: TypeScript, Env, React
   - Remove React imports (injected via CDN)

2. **Launch Browser**: Start Puppeteer with 1280x720 viewport

3. **Measure Iterations**: For each iteration:
   - Set HTML content with React 18 and web-vitals from CDN
   - Wrap component in React.Profiler
   - Wait for initial render
   - Simulate user interactions (clicks on `<a>` and `<button>` elements)
   - Background page (1 second) to trigger Web Vitals calculation
   - Collect evaluation results

4. **Aggregate**: Combine metrics across all iterations

5. **Return**: Performance results with arrays of values

**Usage Example**:

```typescript
import { measurePerformance } from 'react-mcp-server/src/tools/runtimePerf';

const code = `
  function App() {
    const [count, setCount] = React.useState(0);

    return (
      <div>
        <h1>Counter: {count}</h1>
        <button onClick={() => setCount(count + 1)}>
          Increment
        </button>
      </div>
    );
  }
`;

const results = await measurePerformance(code, 3);

console.log('Render times:', results.renderTime);
console.log('LCP values:', results.webVitals.lcp);
console.log('Actual durations:', results.reactProfiler.actualDuration);
```

**Error Handling**:
- Throws Error if code fails to parse
- Throws Error if code fails to transpile
- Sets `results.error` if runtime errors occur during measurement

---

## Performance Results Type

Aggregated performance metrics from multiple measurement iterations.

```typescript { .api }
/**
 * Performance measurement results
 * Contains arrays of metrics collected across iterations
 * Note: This type is NOT exported - it's an internal type shown here for documentation purposes
 */
interface PerformanceResults {
  /** Render time measurements in milliseconds */
  renderTime: number[];

  /** Web Vitals metrics */
  webVitals: {
    /** Cumulative Layout Shift (visual stability) */
    cls: number[];
    /** Largest Contentful Paint in milliseconds (loading speed) */
    lcp: number[];
    /** Interaction to Next Paint in milliseconds (responsiveness) */
    inp: number[];
    /** First Input Delay in milliseconds */
    fid: number[];
    /** Time to First Byte in milliseconds */
    ttfb: number[];
  };

  /** React Profiler metrics */
  reactProfiler: {
    /** Profiler ID (typically 'App') */
    id: number[];
    /** Render phase (mount or update) */
    phase: number[];
    /** Time spent rendering in milliseconds */
    actualDuration: number[];
    /** Estimated time without memoization in milliseconds */
    baseDuration: number[];
    /** When render began (timestamp) */
    startTime: number[];
    /** When update committed (timestamp) */
    commitTime: number[];
  };

  /** Runtime error if any occurred during measurement */
  error: Error | null;
}
```

**Metric Descriptions**:

**Web Vitals**:
- **CLS (Cumulative Layout Shift)**: Measures visual stability
  - Good: ≤ 0.10
  - Needs improvement: 0.10-0.25
  - Poor: > 0.25

- **LCP (Largest Contentful Paint)**: Measures loading speed
  - Good: ≤ 2500ms
  - Needs improvement: 2500-4000ms
  - Poor: > 4000ms

- **INP (Interaction to Next Paint)**: Measures input responsiveness
  - Good: ≤ 200ms
  - Needs improvement: 200-500ms
  - Poor: > 500ms

- **FID (First Input Delay)**: Measures first interaction response time

- **TTFB (Time to First Byte)**: Measures server response time
  - Good: ≤ 800ms

**React Profiler**:
- **actualDuration**: Time spent rendering (measures actual work)
- **baseDuration**: Estimated time without memoization (measures potential work)
- **startTime**: When render phase began
- **commitTime**: When changes were committed to DOM
- **phase**: Either 'mount' (0) or 'update' (1)

**Calculating Mean Values**:

```typescript
function calculateMean(values: number[]): number {
  return values.length > 0
    ? values.reduce((acc, curr) => acc + curr, 0) / values.length
    : 0;
}

const meanLCP = calculateMean(results.webVitals.lcp);
const meanActualDuration = calculateMean(results.reactProfiler.actualDuration);
```

---

## Evaluation Results Type

Single iteration measurement results (internal type).

```typescript { .api }
/**
 * Performance results from a single measurement iteration
 * Used internally during measurement collection
 * Note: This type is NOT exported - it's an internal type shown here for documentation purposes
 */
interface EvaluationResults {
  /** Render time in milliseconds, or null if not collected */
  renderTime: number | null;

  /** Web Vitals metrics (null if not collected) */
  webVitals: {
    cls: number | null;
    lcp: number | null;
    inp: number | null;
    fid: number | null;
    ttfb: number | null;
  };

  /** React Profiler metrics (null if not collected) */
  reactProfiler: {
    id: number | null;
    phase: number | null;
    actualDuration: number | null;
    baseDuration: number | null;
    startTime: number | null;
    commitTime: number | null;
  };

  /** Runtime error if any occurred */
  error: Error | null;
}
```

**Usage**: This type represents the structure of `window.__RESULT__` in the measurement page. Each metric is nullable since it may not be collected in every measurement.

---

## Helper Functions

Utility functions used internally by the performance measurement tool.

### Delay Function

```typescript { .api }
/**
 * Async delay utility
 * @param time - Milliseconds to wait
 * @returns Promise that resolves after the specified time
 */
function delay(time: number): Promise<void>;
```

**Usage**:
```typescript
// Wait 500ms between operations
await delay(500);
```

### Build HTML Function

```typescript { .api }
/**
 * Generate test HTML page with React and web-vitals
 * @param transpiled - Transpiled component code
 * @returns Complete HTML document as string
 */
function buildHtml(transpiled: string): string;
```

**Generated HTML Structure**:
- Loads React 18 from unpkg CDN (`react.development.js`, `react-dom.development.js`)
- Loads web-vitals 3.0.0 from unpkg CDN
- Creates `window.__RESULT__` object for metrics collection
- Sets up web-vitals listeners (onCLS, onLCP, onINP, onFID, onTTFB)
- Injects transpiled component code
- Wraps App component in React.Profiler
- Renders to `<div id="root">`
- Captures render time with `performance.now()`
- Handles uncaught errors and window.onerror

**CDN Resources Used**:
```html
<script src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/web-vitals@3.0.0/dist/web-vitals.iife.js"></script>
```

---

## Babel Configuration

Babel transformation configuration for code transpilation.

```typescript { .api }
/**
 * Babel options for transpiling React components
 * Note: Uses babel.TransformOptions from @babel/core
 */
type BabelOptions = babel.TransformOptions;
```

**Default Configuration**:
```typescript
const babelOptions = {
  filename: 'anonymous.tsx',
  configFile: false,
  babelrc: false,
  presets: [
    babelPresetTypescript,
    babelPresetEnv,
    babelPresetReact
  ]
};
```

**Import Removal Plugin**:

The measurement tool includes a custom Babel plugin to remove React imports (since React is loaded via CDN):

```typescript
{
  plugins: [
    () => ({
      visitor: {
        ImportDeclaration(path) {
          const value = path.node.source.value;
          if (value === 'react' || value === 'react-dom') {
            path.remove();
          }
        }
      }
    })
  ]
}
```

---

## User Interaction Simulation

The measurement tool simulates user interactions to trigger INP and FID metrics.

**Interaction Process**:

1. **Find Interactive Elements**:
   ```javascript
   const elements = Array.from(document.querySelectorAll('a'))
     .concat(Array.from(document.querySelectorAll('button')));
   ```

2. **Collect Selectors**:
   ```javascript
   window.__INTERACTABLE_SELECTORS__ = elements.map(
     el => el.tagName.toLowerCase()
   );
   ```

3. **Click All Elements**:
   ```typescript
   await Promise.all(
     selectors.map(async (selector) => {
       try {
         await page.click(selector);
       } catch (e) {
         console.log(`warning: Could not click ${selector}`);
       }
     })
   );
   ```

4. **Wait for Interactions**: `await delay(500);`

**Elements Targeted**:
- All `<a>` (anchor) elements
- All `<button>` elements

---

## Dependencies

Required packages for performance measurement.

```typescript { .api }
/**
 * Required dependencies
 */
import * as babel from '@babel/core';
import puppeteer from 'puppeteer';
import * as babelPresetTypescript from '@babel/preset-typescript';
import * as babelPresetEnv from '@babel/preset-env';
import * as babelPresetReact from '@babel/preset-react';
```

**Package Versions**:
- `@babel/core@^7.26.0`
- `@babel/preset-env@^7.26.9`
- `@babel/preset-react@^7.18.6`
- `@babel/preset-typescript@^7.27.1`
- `puppeteer@^24.7.2`
