# MCP Tools

The React MCP Server exposes four tools and one prompt through the Model Context Protocol for use by AI assistants.

## Capabilities

### Query React Documentation

Search official React documentation from react.dev using Algolia search.

```typescript { .api }
/**
 * Tool: query-react-dev-docs
 * Searches the official React documentation and returns converted text content
 */
interface QueryReactDevDocsInput {
  /** Search query string */
  query: string;
}

interface MCPToolResponse {
  content: Array<{ type: 'text'; text: string }>;
  isError?: boolean;
}
```

**Description**: This tool searches the React.dev documentation using Algolia's search index and returns the full text content of matching pages.

**Search Behavior**:
- Searches Algolia's 'beta-react' index (appId: '1FCF9AYYAT')
- Returns up to 30 hits per search
- Deduplicates results by URL pathname (ignoring hash fragments)
- Fetches full HTML content from each unique documentation page
- Extracts the `<article>` element (main documentation content)
- Converts HTML to plain text using html-to-text
- Returns empty array if no matches found

**Usage Example**:
```typescript
// When used via MCP protocol
{
  "tool": "query-react-dev-docs",
  "arguments": {
    "query": "useOptimistic hook"
  }
}
```

**Response Format**:
- Success: Array of text content blocks from matching documentation pages
- No results: Single text block with "No results"
- Error: isError flag set to true with error message

---

### Compile React Code

Compile React code with React Compiler for automatic memoization and optimization.

```typescript { .api }
/**
 * Tool: compile
 * Compiles React components and hooks using React Compiler
 */
interface CompileInput {
  /** React component or hook source code to compile */
  text: string;
  /** Optional compiler pass to return intermediate representation */
  passName?: 'HIR' | 'ReactiveFunction' | 'All' | '@DEBUG';
}

interface CompileResponse {
  content: Array<{ type: 'text'; text: string }>;
  isError?: boolean;
}
```

**Description**: Compiles React code through the React Compiler, returning optimized JavaScript/TypeScript with automatic memoization. Can optionally return intermediate compiler passes for debugging.

**Compiler Passes**:
- `HIR` - High-level Intermediate Representation (PropagateScopeDependenciesHIR pass)
- `ReactiveFunction` - Reactive function representation (PruneHoistedContexts pass)
- `All` - Both HIR and ReactiveFunction passes
- `@DEBUG` - All compiler passes with names included

**Compilation Process**:
1. Parse code with Babel (supports TypeScript and JSX)
2. Transform with React Compiler plugin
3. Format output with Prettier (babel-ts parser)
4. Return compiled code or diagnostic messages

**Success Indicators**:
- Import added: `import { c as _c } from "react/compiler-runtime"`
- Cache initialization: `const $ = _c(n)` where n is cache size

**Diagnostic Messages**:
When compilation fails, returns bailout messages with location information:
- Format: `React Compiler bailed out: {message}@{startLine}:{endLine}`
- Common bailouts include manual memoization conflicts, Rules of React violations

**Usage Example**:
```typescript
// Basic compilation
{
  "tool": "compile",
  "arguments": {
    "text": "function MyComponent() { return <div>Hello</div>; }"
  }
}

// With compiler pass debugging
{
  "tool": "compile",
  "arguments": {
    "text": "function MyComponent() { return <div>Hello</div>; }",
    "passName": "All"
  }
}
```

**Response Format**:
- Success: Compiled code as first text block, optional compiler passes as additional blocks
- Bailout: Diagnostic messages with location information
- Error: isError flag set to true with error stack trace

---

### Review Runtime Performance

Measure runtime performance of React components using Web Vitals and React Profiler.

```typescript { .api }
/**
 * Tool: review-react-runtime
 * Measures React component performance across multiple iterations
 */
interface ReviewReactRuntimeInput {
  /**
   * React component code to measure
   * Must contain function App() (not arrow function)
   * No exports allowed
   * Only import React, use React. prefix for hooks
   */
  text: string;
  /** Number of measurement iterations (default: 2) */
  iterations?: number;
}

interface PerformanceResponse {
  content: [{ type: 'text'; text: string }];
  isError?: boolean;
}
```

**Description**: Measures runtime performance by transpiling the code, rendering it in a headless browser, simulating user interactions, and collecting Web Vitals and React Profiler metrics.

**Code Requirements**:
- Must contain `function App()` declaration (not arrow function)
- No export statements allowed
- Only import React from 'react'
- Use `React.` prefix for all hooks (e.g., `React.useState`, `React.useEffect`)

**Performance Metrics Returned**:

**Web Vitals**:
- `CLS` (Cumulative Layout Shift): Visual stability
  - Good: ≤ 0.10, Needs improvement: 0.10-0.25, Poor: > 0.25
- `LCP` (Largest Contentful Paint): Loading speed in ms
  - Good: ≤ 2500ms, Needs improvement: 2500-4000ms, Poor: > 4000ms
- `INP` (Interaction to Next Paint): Input responsiveness in ms
  - Good: ≤ 200ms, Needs improvement: 200-500ms, Poor: > 500ms
- `FID` (First Input Delay): First input response time in ms
- `TTFB` (Time to First Byte): Server response time in ms

**React Profiler**:
- `actualDuration`: Time spent rendering in ms
- `baseDuration`: Estimated time without memoization in ms
- `startTime`: When render began (timestamp)
- `commitTime`: When update committed (timestamp)

**Measurement Process**:
1. Transpile code with Babel (removes React imports, adds preset-env/react/typescript)
2. Launch Puppeteer browser with 1280x720 viewport
3. For each iteration:
   - Set HTML content with React 18 and web-vitals from CDN
   - Wrap component in React.Profiler
   - Wait for initial render
   - Simulate user interactions (clicks on `<a>` and `<button>` elements)
   - Background page to trigger Web Vitals calculation
   - Collect evaluation results
4. Aggregate metrics across iterations (returns mean values)
5. Close browser

**Usage Example**:
```typescript
{
  "tool": "review-react-runtime",
  "arguments": {
    "text": `
      function App() {
        const [count, setCount] = React.useState(0);
        return (
          <div>
            <button onClick={() => setCount(count + 1)}>
              Count: {count}
            </button>
          </div>
        );
      }
    `,
    "iterations": 3
  }
}
```

**Response Format**:
Returns formatted markdown with mean values:
```
# React Component Performance Results

## Mean Render Time
{value}ms

## Mean Web Vitals
- Cumulative Layout Shift (CLS): {value}ms
- Largest Contentful Paint (LCP): {value}ms
- Interaction to Next Paint (INP): {value}ms

## Mean React Profiler
- Actual Duration: {value}ms
- Base Duration: {value}ms
```

---

### Parse Component Tree

Extract the React component tree from a running application using Chrome DevTools Protocol.

```typescript { .api }
/**
 * Tool: parse-react-component-tree
 * Extracts component tree from running React app via Chrome DevTools
 */
interface ParseReactComponentTreeInput {
  /** Application URL (default: 'http://localhost:3000') */
  url?: string;
}

interface ParseComponentTreeResponse {
  content: [{ type: 'text'; text: string }];
  isError?: boolean;
}
```

**Description**: Connects to a Chrome browser running in debug mode and extracts the React component tree structure from a running application using the React DevTools hook.

**Requirements**:
- Chrome browser running on debug port 9222
- React application running at specified URL
- React DevTools hook must be available in the page

**Browser Setup Commands**:
- **macOS**: `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 --user-data-dir=/tmp/chrome`
- **Windows**: `chrome.exe --remote-debugging-port=9222 --user-data-dir=C:\temp\chrome`

**Extraction Process**:
1. Connect to Puppeteer at http://127.0.0.1:9222
2. Find page matching the provided URL (checks if `page.url().startsWith(url)`)
3. Evaluate in page context: `window.__REACT_DEVTOOLS_GLOBAL_HOOK__.rendererInterfaces.get(1).__internal_only_getComponentTree()`
4. Return component tree as string

**Usage Example**:
```typescript
// Default URL
{
  "tool": "parse-react-component-tree",
  "arguments": {}
}

// Custom URL
{
  "tool": "parse-react-component-tree",
  "arguments": {
    "url": "http://localhost:8080/app"
  }
}
```

**Response Format**:
- Success: Component tree structure as JSON string
- Error: Error message if page not found or extraction fails

---

### Code Review Prompt

System prompt providing comprehensive React optimization guidelines for AI assistants.

```typescript { .api }
/**
 * Prompt: review-react-code
 * Returns comprehensive React best practices and optimization guidelines
 */
interface ReviewReactCodePrompt {
  messages: [{
    role: 'assistant';
    content: {
      type: 'text';
      text: string;  // Comprehensive React guidelines
    };
  }];
}
```

**Description**: Provides a detailed system prompt with React optimization guidelines, best practices, and instructions for using the MCP tools effectively.

**Guideline Categories**:
1. **Functional Components and Hooks**: Use functional components, manage state with useState/useReducer
2. **Pure Components**: Keep components pure, avoid side effects during rendering
3. **Data Flow**: Respect one-way data flow, pass data through props
4. **State Updates**: Never mutate state directly, always use immutable updates
5. **useEffect Usage**: Use effects for synchronization only, avoid unnecessary effects
6. **Rules of Hooks**: Call hooks unconditionally at top level
7. **Ref Usage**: Use refs only when necessary, avoid during rendering
8. **Component Composition**: Prefer composition and small components
9. **Concurrency**: Write code compatible with React's concurrent rendering
10. **Network Optimization**: Use parallel data fetching, avoid waterfalls
11. **React Compiler**: Rely on React Compiler instead of manual memoization
12. **User Experience**: Provide clear loading states, handle errors gracefully
13. **Server Components**: Shift data-heavy logic to server when possible

**Tool References**:
- `docs` tool: Look up documentation from react.dev
- `compile` tool: Run code through React Compiler

**Code Review Process**:
1. Analyze code for optimization opportunities
2. Use React Compiler to verify optimization potential
3. Provide actionable guidance with before/after examples

**Understanding Compiler Output**:
- Successful optimization adds: `import { c as _c } from "react/compiler-runtime"`
- Cache initialization: `const $ = _c(n)` where n is cache size
- Increasing n: More memoization coverage
- Decreasing n: Fewer dependencies, less re-rendering

**Usage**: This prompt is automatically available to AI assistants connected to the MCP server and can be invoked to provide React optimization context.
