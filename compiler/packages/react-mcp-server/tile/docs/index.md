# React MCP Server

An experimental Model Context Protocol (MCP) server that provides React development tools and utilities for AI-powered coding assistants. The server integrates React Compiler, performance measurement, documentation search, and component tree inspection capabilities.

## Package Information

- **Package Name**: react-mcp-server
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: `npm install react-mcp-server`
- **Binary**: `react-mcp-server` (after installation)

## Core Imports

```typescript
// Main MCP server components are not directly importable
// The package is used as an MCP server binary
```

For programmatic access to individual modules:

```typescript
import { compile, lastResult, type PrintedCompilerPipelineValue } from 'react-mcp-server/src/compiler';
import { parseReactComponentTree } from 'react-mcp-server/src/tools/componentTree';
import { measurePerformance } from 'react-mcp-server/src/tools/runtimePerf';
import { queryAlgolia, printHierarchy, ALGOLIA_CLIENT } from 'react-mcp-server/src/utils/algolia';
import assertExhaustive from 'react-mcp-server/src/utils/assertExhaustive';
import type { DocSearchHit, InternalDocSearchHit } from 'react-mcp-server/src/types/algolia';
```

## Basic Usage

This package is designed to run as an MCP server for AI assistants like Claude Desktop. Configure in `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "react": {
      "command": "/path/to/node",
      "args": [
        "/path/to/react-mcp-server/dist/index.js"
      ]
    }
  }
}
```

Once connected, the AI assistant can access four MCP tools:
- `query-react-dev-docs` - Search React documentation
- `compile` - Compile code with React Compiler
- `review-react-runtime` - Measure runtime performance
- `parse-react-component-tree` - Extract component tree from running apps

## Architecture

The React MCP Server is built on the Model Context Protocol SDK and provides:

**MCP Layer**: Server instance with stdio transport exposing tools and prompts to AI assistants

**Tool Layer**: Four specialized tools for React development workflows:
- Documentation search via Algolia
- React Compiler integration with diagnostic reporting
- Runtime performance measurement using Web Vitals and React Profiler
- Component tree extraction via Chrome DevTools Protocol

**Core Modules**:
- Compiler module: React Compiler integration with Babel and Prettier
- Performance module: Puppeteer-based runtime measurement
- Component tree module: React DevTools integration
- Algolia module: Documentation search and retrieval

## Capabilities

### MCP Tools

The server exposes four MCP tools for React development assistance.

```typescript { .api }
// Tool: query-react-dev-docs
interface QueryReactDevDocsParams {
  query: string;  // Search query string
}

// Tool: compile
interface CompileParams {
  text: string;  // React component or hook source code
  passName?: 'HIR' | 'ReactiveFunction' | 'All' | '@DEBUG';  // Optional compiler pass
}

// Tool: review-react-runtime
interface ReviewReactRuntimeParams {
  text: string;  // React component code (must have App functional component)
  iterations?: number;  // Default: 2
}

// Tool: parse-react-component-tree
interface ParseReactComponentTreeParams {
  url?: string;  // Default: 'http://localhost:3000'
}
```

[MCP Tools Documentation](./mcp-tools.md)

### React Compiler Integration

Compile React code with automatic memoization via React Compiler.

```typescript { .api }
function compile(options: {
  text: string;
  file: string;
  options: PluginOptions | null;
}): Promise<BabelFileResult>;

let lastResult: BabelFileResult | null;

type PrintedCompilerPipelineValue =
  | { kind: 'hir'; name: string; fnName: string | null; value: string }
  | { kind: 'reactive'; name: string; fnName: string | null; value: string }
  | { kind: 'debug'; name: string; fnName: string | null; value: string };
```

[Compiler API Documentation](./compiler-api.md)

### Runtime Performance Measurement

Measure React component performance with Web Vitals and React Profiler.

```typescript { .api }
function measurePerformance(
  code: string,
  iterations: number
): Promise<PerformanceResults>;

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

[Performance API Documentation](./performance-api.md)

### Component Tree Inspection

Extract component tree from running React applications.

```typescript { .api }
function parseReactComponentTree(url: string): Promise<string>;
```

[Component Tree API Documentation](./component-tree-api.md)

### React Documentation Search

Search official React documentation via Algolia.

```typescript { .api }
function queryAlgolia(message: string | string[]): Promise<string[]>;

function printHierarchy(hit: DocSearchHit | InternalDocSearchHit): string;

const ALGOLIA_CLIENT: AlgoliaClient;
```

[Algolia API Documentation](./algolia-api.md)

### Utility Functions

Helper functions for development.

```typescript { .api }
function assertExhaustive(_: never, errorMsg: string): never;
```

[Utilities API Documentation](./utils-api.md)

## Types

Core type definitions used across the API.

```typescript { .api }
type ContentType = 'content' | 'lvl0' | 'lvl1' | 'lvl2' | 'lvl3' | 'lvl4' | 'lvl5' | 'lvl6';

interface DocSearchHit {
  objectID: string;
  content: string | null;
  url: string;
  url_without_anchor: string;
  type: ContentType;
  anchor: string | null;
  hierarchy: {
    lvl0: string;
    lvl1: string;
    lvl2: string | null;
    lvl3: string | null;
    lvl4: string | null;
    lvl5: string | null;
    lvl6: string | null;
  };
  _highlightResult: DocSearchHitHighlightResult;
  _snippetResult: DocSearchHitSnippetResult;
  _rankingInfo?: RankingInfo;
  _distinctSeqID?: number;
  __autocomplete_indexName?: string;
  __autocomplete_queryID?: string;
  __autocomplete_algoliaCredentials?: AlgoliaCredentials;
  __autocomplete_id?: number;
}

type InternalDocSearchHit = DocSearchHit & {
  __docsearch_parent: InternalDocSearchHit | null;
};

interface DocSearchHitHighlightResult {
  content: DocSearchHitAttributeHighlightResult;
  hierarchy: DocSearchHitHighlightResultHierarchy;
  hierarchy_camel: DocSearchHitHighlightResultHierarchy[];
}

interface DocSearchHitAttributeHighlightResult {
  value: string;
  matchLevel: 'full' | 'none' | 'partial';
  matchedWords: string[];
  fullyHighlighted?: boolean;
}

interface DocSearchHitHighlightResultHierarchy {
  lvl0: DocSearchHitAttributeHighlightResult;
  lvl1: DocSearchHitAttributeHighlightResult;
  lvl2: DocSearchHitAttributeHighlightResult;
  lvl3: DocSearchHitAttributeHighlightResult;
  lvl4: DocSearchHitAttributeHighlightResult;
  lvl5: DocSearchHitAttributeHighlightResult;
  lvl6: DocSearchHitAttributeHighlightResult;
}

interface DocSearchHitSnippetResult {
  content: DocSearchHitAttributeSnippetResult;
  hierarchy: DocSearchHitHighlightResultHierarchy;
  hierarchy_camel: DocSearchHitHighlightResultHierarchy[];
}

interface DocSearchHitAttributeSnippetResult {
  value: string;
  matchLevel: 'full' | 'none' | 'partial';
}

interface RankingInfo {
  promoted: boolean;
  nbTypos: number;
  firstMatchedWord: number;
  proximityDistance?: number;
  geoDistance: number;
  geoPrecision?: number;
  nbExactWords: number;
  words: number;
  filters: number;
  userScore: number;
  matchedGeoLocation?: {
    lat: number;
    lng: number;
    distance: number;
  };
}

interface AlgoliaCredentials {
  appId: string;
  apiKey: string;
}
```
