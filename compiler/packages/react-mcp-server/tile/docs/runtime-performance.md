# React Runtime Performance

Measure runtime performance of React components using Web Vitals metrics and React Profiler data. This tool runs React components in a headless browser and collects performance metrics to verify optimization improvements.

## Capabilities

### Review React Runtime Tool

Measure the runtime performance of React components by executing them in a Puppeteer-controlled headless browser. Collects Web Vitals metrics (LCP, INP, CLS) and React Profiler data.

```typescript { .api }
/**
 * Measure runtime performance of React components
 *
 * This tool runs React components in a headless browser and collects:
 * - Web Vitals metrics (LCP, INP, CLS, FID, TTFB)
 * - React Profiler metrics (actualDuration, baseDuration, etc.)
 * - Render time measurements
 *
 * Use this tool to verify that code optimizations actually improve performance.
 * Always run on original code first, then on modified code to compare results.
 *
 * Code Requirements:
 * - MUST contain an App functional component (not arrow function)
 * - DO NOT export anything (exports cannot be parsed)
 * - Only import React from 'react' and use React. prefix (React.useState, React.useEffect)
 *
 * Performance Goals:
 * - LCP (loading speed): good ≤ 2.5s, needs-improvement 2.5-4s, poor > 4s
 * - INP (input responsiveness): good ≤ 200ms, needs-improvement 200-500ms, poor > 500ms
 * - CLS (visual stability): good ≤ 0.10, needs-improvement 0.10-0.25, poor > 0.25
 *
 * @param text - React component code with App functional component
 * @param iterations - Number of measurement iterations (default: 2)
 * @returns Performance metrics including render time, Web Vitals, and React Profiler data
 */
interface ReviewReactRuntimeTool {
  name: 'review-react-runtime';
  description: 'Run this tool every time you propose a performance related change to verify if your suggestion actually improves performance.';
  inputSchema: {
    text: string;
    iterations?: number; // default: 2
  };
}
```

**Parameters:**

```typescript { .api }
interface ReviewReactRuntimeParams {
  /**
   * React component code to measure
   *
   * Requirements:
   * - MUST contain an App functional component (not arrow function):
   *   ✅ function App() { ... }
   *   ❌ const App = () => { ... }
   * - DO NOT export anything (no export, export default, etc.)
   * - Only import React and use React. prefix:
   *   ✅ React.useState, React.useEffect, React.memo
   *   ❌ import { useState } from 'react'
   */
  text: string;

  /**
   * Number of performance measurement iterations
   * Higher iterations provide more accurate average results
   * Default: 2
   */
  iterations?: number;
}
```

**Response Format:**

```typescript { .api }
interface PerformanceResponse {
  content: [{
    type: 'text';
    text: string; // Formatted performance results
  }];
  isError?: boolean;
}
```

Successful measurement returns formatted results:
```
# React Component Performance Results

## Mean Render Time
<mean>ms

## Mean Web Vitals
- Cumulative Layout Shift (CLS): <mean>ms
- Largest Contentful Paint (LCP): <mean>ms
- Interaction to Next Paint (INP): <mean>ms

## Mean React Profiler
- Actual Duration: <mean>ms
- Base Duration: <mean>ms
```

Error returns:
```typescript
{
  isError: true,
  content: [{
    type: 'text',
    text: 'Error measuring performance: <error message>\n\n<stack trace>'
  }]
}
```

## Performance Metrics

### Web Vitals

```typescript { .api }
interface WebVitalsMetrics {
  /** Cumulative Layout Shift - visual stability metric */
  cls: number[];

  /** Largest Contentful Paint - loading performance metric (ms) */
  lcp: number[];

  /** Interaction to Next Paint - input responsiveness metric (ms) */
  inp: number[];

  /** First Input Delay - initial interactivity metric (ms) */
  fid: number[];

  /** Time to First Byte - server response time metric (ms) */
  ttfb: number[];
}
```

**Thresholds:**
- **LCP** (loading speed):
  - Good: ≤ 2.5s
  - Needs improvement: 2.5-4s
  - Poor: > 4s
- **INP** (input responsiveness):
  - Good: ≤ 200ms
  - Needs improvement: 200-500ms
  - Poor: > 500ms
- **CLS** (visual stability):
  - Good: ≤ 0.10
  - Needs improvement: 0.10-0.25
  - Poor: > 0.25

### React Profiler

```typescript { .api }
interface ReactProfilerMetrics {
  /** Component ID */
  id: number[];

  /** Render phase */
  phase: number[];

  /** Actual time spent rendering (ms) */
  actualDuration: number[];

  /** Estimated time to render entire subtree without memoization (ms) */
  baseDuration: number[];

  /** When React began rendering (timestamp) */
  startTime: number[];

  /** When React committed the update (timestamp) */
  commitTime: number[];
}
```

### Render Time

```typescript { .api }
interface RenderTimeMetrics {
  /** Total render time in milliseconds */
  renderTime: number[];
}
```

## Usage Examples

### Measuring Original Code

```typescript
// Measure baseline performance before optimization
{
  tool: 'review-react-runtime',
  params: {
    text: `
function App() {
  const [count, setCount] = React.useState(0);
  const [items, setItems] = React.useState([]);

  React.useEffect(() => {
    // Expensive operation on every render
    const result = items.map(item => ({
      ...item,
      processed: true
    }));
    console.log(result);
  }, [items]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      <button onClick={() => setItems([...items, { id: Date.now() }])}>
        Add Item
      </button>
    </div>
  );
}
    `,
    iterations: 3
  }
}
```

### Measuring Optimized Code

```typescript
// Measure performance after optimization
{
  tool: 'review-react-runtime',
  params: {
    text: `
function App() {
  const [count, setCount] = React.useState(0);
  const [items, setItems] = React.useState([]);

  React.useEffect(() => {
    // Fixed: moved expensive operation outside effect
    const result = items.map(item => ({
      ...item,
      processed: true
    }));
    console.log(result);
  }, [items]); // Now only runs when items change

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      <button onClick={() => setItems([...items, { id: Date.now() }])}>
        Add Item
      </button>
    </div>
  );
}
    `,
    iterations: 3
  }
}
```

### Code Requirements Example

```typescript
// ✅ CORRECT format
{
  tool: 'review-react-runtime',
  params: {
    text: `
function App() {
  const [value, setValue] = React.useState(0);

  return (
    <div>
      <button onClick={() => setValue(value + 1)}>
        {value}
      </button>
    </div>
  );
}
    `
  }
}

// ❌ INCORRECT - arrow function
{
  tool: 'review-react-runtime',
  params: {
    text: `
const App = () => { // ❌ Must be function declaration
  return <div>Hello</div>;
}
    `
  }
}

// ❌ INCORRECT - exports
{
  tool: 'review-react-runtime',
  params: {
    text: `
export default function App() { // ❌ No exports allowed
  return <div>Hello</div>;
}
    `
  }
}

// ❌ INCORRECT - destructured imports
{
  tool: 'review-react-runtime',
  params: {
    text: `
import { useState } from 'react'; // ❌ Must use React. prefix

function App() {
  const [value] = useState(0); // ❌ Must be React.useState
  return <div>{value}</div>;
}
    `
  }
}
```

## Optimization Workflow

### Iterative Performance Improvement

```
1. Measure baseline (original code)
   └─> Run review-react-runtime with original code

2. Identify worst metric
   └─> Check which metric is "poor" or "needs-improvement"

3. Apply targeted optimization
   ├─> LCP issues: lazy-load images, inline critical CSS, React.lazy + Suspense
   ├─> INP issues: wrap updates in useTransition, avoid setState in useEffect
   └─> CLS issues: reserve space, stable keys, fixed-size skeletons

4. Measure improvement (optimized code)
   └─> Run review-react-runtime with optimized code

5. Compare results
   └─> Verify metrics improved from baseline

6. Repeat until all metrics are "good" or two consecutive cycles show no gain
```

### Example: Fixing LCP (Loading Performance)

```typescript
// Step 1: Measure baseline
{
  tool: 'review-react-runtime',
  params: {
    text: `
function App() {
  return (
    <div>
      <img src="/large-hero.jpg" alt="Hero" />
      <Component1 />
      <Component2 />
      <Component3 />
    </div>
  );
}
    `
  }
}
// Result: LCP = 4.2s (poor)

// Step 2: Apply React.lazy for below-the-fold components
{
  tool: 'review-react-runtime',
  params: {
    text: `
const Component2 = React.lazy(() => import('./Component2'));
const Component3 = React.lazy(() => import('./Component3'));

function App() {
  return (
    <div>
      <img src="/large-hero.jpg" alt="Hero" />
      <Component1 />
      <React.Suspense fallback={<div>Loading...</div>}>
        <Component2 />
        <Component3 />
      </React.Suspense>
    </div>
  );
}
    `
  }
}
// Result: LCP = 2.1s (good) ✅
```

### Example: Fixing INP (Interactivity)

```typescript
// Step 1: Measure baseline
{
  tool: 'review-react-runtime',
  params: {
    text: `
function App() {
  const [items, setItems] = React.useState([]);

  const handleAdd = () => {
    // Expensive synchronous update blocks interactions
    const newItems = [...items, ...Array(1000).fill(0).map((_, i) => ({
      id: i,
      value: Math.random()
    }))];
    setItems(newItems);
  };

  return (
    <div>
      <button onClick={handleAdd}>Add Items</button>
      {items.map(item => <div key={item.id}>{item.value}</div>)}
    </div>
  );
}
    `
  }
}
// Result: INP = 450ms (needs improvement)

// Step 2: Wrap in useTransition for non-blocking update
{
  tool: 'review-react-runtime',
  params: {
    text: `
function App() {
  const [items, setItems] = React.useState([]);
  const [isPending, startTransition] = React.useTransition();

  const handleAdd = () => {
    startTransition(() => {
      // Update now non-blocking
      const newItems = [...items, ...Array(1000).fill(0).map((_, i) => ({
        id: i,
        value: Math.random()
      }))];
      setItems(newItems);
    });
  };

  return (
    <div>
      <button onClick={handleAdd}>
        Add Items {isPending && '(pending...)'}
      </button>
      {items.map(item => <div key={item.id}>{item.value}</div>)}
    </div>
  );
}
    `
  }
}
// Result: INP = 180ms (good) ✅
```

## Implementation Details

The tool:
1. Transpiles React code using Babel (TypeScript, JSX support)
2. Removes React imports (provided via CDN in test environment)
3. Launches Puppeteer headless browser
4. Creates HTML page with React 18 UMD and web-vitals library
5. Executes code in browser with React Profiler wrapper
6. Simulates user interactions (clicks on buttons and links)
7. Collects Web Vitals metrics after page backgrounds
8. Aggregates metrics across iterations
9. Returns mean values for all metrics

## Best Practices

1. **Always measure baseline first:** Run on original code before making changes
2. **Use sufficient iterations:** Higher iteration count (3-5) provides more reliable results
3. **Compare results:** Verify optimizations actually improve metrics
4. **Focus on worst metric:** Optimize the poorest-performing metric first
5. **Follow React patterns:** Use useTransition, React.lazy, Suspense appropriately
6. **Avoid premature optimization:** Only optimize after measuring actual performance issues
7. **Verify with compile tool:** Combine with React Compiler for automatic memoization

## When to Use

- Verifying performance improvements from code changes
- Comparing optimized vs. unoptimized code
- Measuring impact of React Compiler optimizations
- Identifying performance bottlenecks in React components
- Validating Web Vitals metrics meet performance goals
- Testing effect of useTransition, React.lazy, and other optimizations
