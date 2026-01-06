# React MCP Server

React MCP Server is an experimental Model Context Protocol (MCP) server that provides React development tools for AI assistants and development environments. It enables querying official React documentation, compiling React code with React Compiler for automatic memoization, measuring runtime performance with Web Vitals, and parsing component trees from running applications.

## Package Information

- **Package Name**: react-mcp-server
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: `npm install react-mcp-server`
- **Binary**: `react-mcp-server`

## Core Setup

### Running the Server

The server runs as an MCP server over stdio:

```bash
node /path/to/react-mcp-server/dist/index.js
```

### MCP Client Configuration

For Claude Desktop, add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "react": {
      "command": "/path/to/node",
      "args": ["/path/to/react-mcp-server/dist/index.js"]
    }
  }
}
```

## Architecture

React MCP Server is built on the Model Context Protocol (MCP) and provides the following components:

- **MCP Tools**: Four tools for React documentation search, code compilation, performance measurement, and component tree inspection
- **MCP Prompts**: System prompt for React code review and optimization guidance
- **Algolia Integration**: Direct access to react.dev documentation via Algolia search
- **React Compiler Integration**: Babel-based compilation with React Compiler plugin
- **Performance Measurement**: Puppeteer-based performance testing with Web Vitals
- **DevTools Integration**: Component tree extraction via Chrome DevTools Protocol

## Capabilities

### React Documentation Search

Query official React documentation from react.dev using Algolia search. Returns formatted documentation content for hooks, components, and APIs.

```typescript { .api }
// MCP Tool: query-react-dev-docs
interface QueryReactDevDocsParams {
  query: string;
}
```

[React Documentation Search](./query-react-docs.md)

### React Code Compilation

Compile React components and hooks using React Compiler to generate automatically memoized code. Returns compiled JavaScript/TypeScript with diagnostic messages for optimization opportunities.

```typescript { .api }
// MCP Tool: compile
interface CompileParams {
  text: string;
  passName?: 'HIR' | 'ReactiveFunction' | 'All' | '@DEBUG';
}
```

[React Code Compilation](./compile.md)

### React Runtime Performance

Measure runtime performance of React components using Web Vitals metrics (LCP, INP, CLS) and React Profiler data. Helps verify performance improvements from optimizations.

```typescript { .api }
// MCP Tool: review-react-runtime
interface ReviewReactRuntimeParams {
  text: string;
  iterations?: number; // default: 2
}
```

[React Runtime Performance](./runtime-performance.md)

### React Component Tree Parsing

Parse the component tree from a running React application via Chrome DevTools Protocol. Requires Chrome running in debug mode.

```typescript { .api }
// MCP Tool: parse-react-component-tree
interface ParseReactComponentTreeParams {
  url?: string; // default: 'http://localhost:3000'
}
```

[React Component Tree Parsing](./component-tree.md)

### React Code Review Prompt

System prompt providing guidelines for reviewing and optimizing React code with React Compiler, including best practices and optimization strategies.

```typescript { .api }
// MCP Prompt: review-react-code
// Returns assistant message with React optimization guidelines
```

[React Code Review Prompt](./code-review-prompt.md)

## Common Types

```typescript { .api }
// MCP SDK types used across tools
interface MCPToolResponse {
  content: Array<{ type: 'text'; text: string }>;
  isError?: boolean;
}

// Compilation types
type PassName = 'HIR' | 'ReactiveFunction' | 'All' | '@DEBUG';

// Performance measurement results
interface PerformanceResults {
  renderTime: number[];
  webVitals: {
    cls: number[];
    lcp: number[];
    inp: number[];
    fid: number[];
    ttfb: number[];
  };
  reactProfiler: {
    id: number[];
    phase: number[];
    actualDuration: number[];
    baseDuration: number[];
    startTime: number[];
    commitTime: number[];
  };
  error: Error | null;
}
```
