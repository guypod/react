# React Documentation Search

Query official React documentation from react.dev using Algolia search. This tool provides access to the most up-to-date React documentation, including APIs, hooks, components, and best practices.

## Capabilities

### Query React Dev Docs Tool

Search for official documentation from react.dev using Algolia. Returns HTML content converted to text from matching documentation pages.

```typescript { .api }
/**
 * Search for official React documentation from react.dev
 *
 * This tool provides access to documentation for React APIs such as:
 * - Components: <ViewTransition>, <Activity>, <Suspense>, <Fragment>, etc.
 * - Hooks: useOptimistic, useSyncExternalStore, useTransition, useState, useEffect, etc.
 * - APIs: createRoot, hydrateRoot, act, etc.
 *
 * The tool queries Algolia's index of react.dev and returns formatted text content
 * from the matching pages, focusing on the main article content.
 *
 * @param query - Search query string (e.g., "useOptimistic", "Suspense", "Server Components")
 * @returns MCP tool response with text content from matching documentation pages
 */
interface QueryReactDevDocsTool {
  name: 'query-react-dev-docs';
  description: 'This tool lets you search for official docs from react.dev. This always has the most up to date information on React. You can look for documentation on APIs such as <ViewTransition>, <Activity>, and hooks like useOptimistic, useSyncExternalStore, useTransition, and more. Whenever you think hard about React, use this tool to get more information before proceeding.';
  inputSchema: {
    query: string;
  };
}
```

**Response Format:**

```typescript { .api }
interface MCPToolResponse {
  content: Array<{
    type: 'text';
    text: string; // HTML converted to plain text from article content
  }>;
  isError?: boolean;
}
```

When no results are found, returns:
```typescript
{
  content: [{ type: 'text', text: 'No results' }]
}
```

When an error occurs, returns:
```typescript
{
  isError: true,
  content: [{ type: 'text', text: 'Error: <error message>' }]
}
```

## Usage Examples

### Searching for Hook Documentation

```typescript
// Search for useOptimistic hook documentation
{
  tool: 'query-react-dev-docs',
  params: {
    query: 'useOptimistic'
  }
}

// Returns formatted text content from react.dev/reference/react/useOptimistic
```

### Searching for Component Documentation

```typescript
// Search for Suspense component documentation
{
  tool: 'query-react-dev-docs',
  params: {
    query: 'Suspense component'
  }
}

// Returns formatted text content from react.dev/reference/react/Suspense
```

### Searching for Concepts

```typescript
// Search for Server Components documentation
{
  tool: 'query-react-dev-docs',
  params: {
    query: 'Server Components'
  }
}

// Returns formatted text content from relevant react.dev pages about Server Components
```

## Implementation Details

The tool:
1. Queries Algolia's `beta-react` index with the provided search term
2. Retrieves up to 30 hits from the search results
3. Deduplicates results by URL pathname (ignoring hash fragments)
4. Fetches the full HTML content from each unique documentation page
5. Extracts the `<article>` element content (main documentation content)
6. Converts HTML to plain text using html-to-text library
7. Returns all matching pages as separate text content items

### Algolia Configuration

```typescript { .api }
// Internal configuration (not directly accessible)
const ALGOLIA_CONFIG = {
  appId: '1FCF9AYYAT',
  apiKey: '1b7ad4e1c89e645e351e59d40544eda1',
  indexName: 'beta-react'
};
```

## When to Use

- Looking up React API documentation
- Finding information about hooks (useState, useEffect, useOptimistic, etc.)
- Researching React components (<Suspense>, <Fragment>, etc.)
- Understanding React concepts (Server Components, Transitions, etc.)
- Getting the most current React best practices
- Verifying React API signatures and usage patterns
