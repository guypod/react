# Algolia API

Search and retrieve React documentation from react.dev using Algolia search.

## Capabilities

### Query Algolia

Search the official React documentation and fetch full page content.

```typescript { .api }
/**
 * Search React documentation via Algolia and fetch full HTML content
 * @param message - Search query string or array of strings
 * @returns Promise resolving to array of HTML page content strings
 */
function queryAlgolia(message: string | string[]): Promise<string[]>;
```

**Search Configuration**:
- **Algolia App ID**: `1FCF9AYYAT`
- **API Key**: `1b7ad4e1c89e645e351e59d40544eda1` (public search-only key)
- **Index Name**: `beta-react`
- **Hits Per Page**: 30

**Search Process**:

1. **Prepare Query**: Join array messages with newlines
   ```typescript
   const query = Array.isArray(message) ? message.join('\n') : message;
   ```

2. **Search Algolia**: Query the beta-react index
   ```typescript
   const { results } = await ALGOLIA_CLIENT.search<DocSearchHit>({
     requests: [{
       query,
       indexName: 'beta-react',
       attributesToRetrieve: [
         'hierarchy.lvl0', 'hierarchy.lvl1', 'hierarchy.lvl2',
         'hierarchy.lvl3', 'hierarchy.lvl4', 'hierarchy.lvl5',
         'hierarchy.lvl6', 'content', 'url'
       ],
       attributesToSnippet: [
         'hierarchy.lvl1:10', 'hierarchy.lvl2:10', 'hierarchy.lvl3:10',
         'hierarchy.lvl4:10', 'hierarchy.lvl5:10', 'hierarchy.lvl6:10',
         'content:10'
       ],
       snippetEllipsisText: '…',
       hitsPerPage: 30,
       attributesToHighlight: [
         'hierarchy.lvl0', 'hierarchy.lvl1', 'hierarchy.lvl2',
         'hierarchy.lvl3', 'hierarchy.lvl4', 'hierarchy.lvl5',
         'hierarchy.lvl6', 'content'
       ]
     }]
   });
   ```

3. **Deduplicate Results**: Remove duplicate URLs (ignoring hash fragments)
   ```typescript
   const deduped = new Map();
   for (const hit of hits) {
     const u = new URL(hit.url);
     if (!deduped.has(u.pathname)) {
       deduped.set(u.pathname, hit);
     }
   }
   ```

4. **Fetch Full Content**: Retrieve complete HTML for each unique URL
   ```typescript
   const pages = await Promise.all(
     Array.from(deduped.values()).map(hit =>
       fetch(hit.url, {
         headers: {
           'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...'
         }
       }).then(res => res.ok ? res.text() : null)
     )
   );
   ```

5. **Filter and Return**: Remove null responses and return HTML strings

**Usage Example**:

```typescript
import { queryAlgolia } from 'react-mcp-server/src/utils/algolia';

// Single query
const pages = await queryAlgolia('useEffect');
console.log(`Found ${pages.length} pages`);

// Multiple terms
const pagesMulti = await queryAlgolia(['hooks', 'useState', 'useRef']);

// Process results
for (const html of pages) {
  // Parse HTML, extract content, etc.
  console.log(`Page length: ${html.length} characters`);
}
```

**Response Format**:
- Array of HTML strings (full page content)
- Empty array if no results found
- Null responses filtered out

---

### Print Hierarchy

Format Algolia search hit hierarchy as a readable string.

```typescript { .api }
/**
 * Format documentation hierarchy from search hit
 * @param hit - Algolia search result hit
 * @returns Formatted hierarchy string (e.g., "API > Hooks > useState")
 */
function printHierarchy(hit: DocSearchHit | InternalDocSearchHit): string;
```

**Output Format**: `lvl0 > lvl1 [> lvl2] [> lvl3] [> lvl4] [> lvl5] [> lvl6]`

**Hierarchy Levels**:
- `lvl0`: Top-level category (e.g., "API Reference")
- `lvl1`: Second-level category (e.g., "Hooks")
- `lvl2`: Third-level category (e.g., "State Hooks")
- `lvl3-lvl6`: Additional nesting levels (optional)

**Usage Example**:

```typescript
import { queryAlgolia, printHierarchy } from 'react-mcp-server/src/utils/algolia';
import { ALGOLIA_CLIENT } from 'react-mcp-server/src/utils/algolia';

// Search and print hierarchies
const { results } = await ALGOLIA_CLIENT.search<DocSearchHit>({
  requests: [{
    query: 'useState',
    indexName: 'beta-react',
    hitsPerPage: 5
  }]
});

const hits = results[0].hits;
for (const hit of hits) {
  console.log(printHierarchy(hit));
  // Output examples:
  // API Reference > Hooks > useState
  // Learn > State Management > Using useState
}
```

**Example Outputs**:
```
API Reference > react > Hooks
API Reference > react > useState
Learn > Managing State > Sharing State Between Components
Reference > React DOM APIs > createRoot
```

---

### Algolia Client

Pre-configured Algolia client instance for React documentation search.

```typescript { .api }
/**
 * Algolia lite client configured for React.dev documentation
 * Uses public search-only API key
 * Type: ReturnType<typeof liteClient> from algoliasearch/lite
 */
const ALGOLIA_CLIENT: ReturnType<typeof import('algoliasearch/lite').liteClient>;
```

**Configuration**:
```typescript
const ALGOLIA_CONFIG = {
  appId: '1FCF9AYYAT',
  apiKey: '1b7ad4e1c89e645e351e59d40544eda1',
  indexName: 'beta-react'
};

const ALGOLIA_CLIENT = liteClient(
  ALGOLIA_CONFIG.appId,
  ALGOLIA_CONFIG.apiKey
);
```

**Usage**:

```typescript
import { ALGOLIA_CLIENT } from 'react-mcp-server/src/utils/algolia';
import type { DocSearchHit } from 'react-mcp-server/src/types/algolia';

// Custom search
const { results } = await ALGOLIA_CLIENT.search<DocSearchHit>({
  requests: [{
    query: 'useTransition',
    indexName: 'beta-react',
    hitsPerPage: 10,
    filters: 'type:lvl1'  // Only level 1 results
  }]
});

// Multiple queries
const { results: multiResults } = await ALGOLIA_CLIENT.search<DocSearchHit>({
  requests: [
    { query: 'hooks', indexName: 'beta-react', hitsPerPage: 5 },
    { query: 'components', indexName: 'beta-react', hitsPerPage: 5 }
  ]
});
```

---

## Highlight and Snippet Types

Types for search result highlighting and snippets.

```typescript { .api }
/**
 * Highlighted attribute in search results
 */
interface DocSearchHitAttributeHighlightResult {
  /** Highlighted value with <mark> tags */
  value: string;
  /** Match quality: full, partial, or none */
  matchLevel: 'full' | 'none' | 'partial';
  /** Matched search terms */
  matchedWords: string[];
  /** Whether entire value was highlighted */
  fullyHighlighted?: boolean;
}

/**
 * Hierarchy highlight results
 */
interface DocSearchHitHighlightResultHierarchy {
  lvl0: DocSearchHitAttributeHighlightResult;
  lvl1: DocSearchHitAttributeHighlightResult;
  lvl2: DocSearchHitAttributeHighlightResult;
  lvl3: DocSearchHitAttributeHighlightResult;
  lvl4: DocSearchHitAttributeHighlightResult;
  lvl5: DocSearchHitAttributeHighlightResult;
  lvl6: DocSearchHitAttributeHighlightResult;
}

/**
 * Complete highlight result for a hit
 */
interface DocSearchHitHighlightResult {
  content: DocSearchHitAttributeHighlightResult;
  hierarchy: DocSearchHitHighlightResultHierarchy;
  hierarchy_camel: DocSearchHitHighlightResultHierarchy[];
}

/**
 * Snippet attribute with context
 */
interface DocSearchHitAttributeSnippetResult {
  /** Snippet text with ellipsis */
  value: string;
  /** Match quality */
  matchLevel: 'full' | 'none' | 'partial';
}

/**
 * Complete snippet result for a hit
 */
interface DocSearchHitSnippetResult {
  content: DocSearchHitAttributeSnippetResult;
  hierarchy: DocSearchHitHighlightResultHierarchy;
  hierarchy_camel: DocSearchHitHighlightResultHierarchy[];
}
```

**Highlight Example**:
```typescript
{
  value: "<mark>useState</mark> is a React Hook",
  matchLevel: "full",
  matchedWords: ["useState"],
  fullyHighlighted: false
}
```

**Snippet Example**:
```typescript
{
  value: "…<mark>useState</mark> lets you add state…",
  matchLevel: "partial"
}
```

---

## DocSearchHit Type

Algolia search result structure for React documentation.

```typescript { .api }
/**
 * Algolia search hit from React documentation index
 */
interface DocSearchHit {
  /** Unique hit identifier */
  objectID: string;

  /** Page content snippet */
  content: string | null;

  /** Full documentation URL */
  url: string;

  /** URL without anchor/hash fragment */
  url_without_anchor: string;

  /** Content type indicator */
  type: 'content' | 'lvl0' | 'lvl1' | 'lvl2' | 'lvl3' | 'lvl4' | 'lvl5' | 'lvl6';

  /** Anchor/hash fragment */
  anchor: string | null;

  /** Documentation hierarchy */
  hierarchy: {
    lvl0: string;
    lvl1: string;
    lvl2: string | null;
    lvl3: string | null;
    lvl4: string | null;
    lvl5: string | null;
    lvl6: string | null;
  };

  /** Highlighted search results */
  _highlightResult: DocSearchHitHighlightResult;

  /** Snippet results with match context */
  _snippetResult: DocSearchHitSnippetResult;

  /** Ranking information (optional) */
  _rankingInfo?: {
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
  };

  /** Distinct sequence ID (optional) */
  _distinctSeqID?: number;

  /** Autocomplete metadata (optional) */
  __autocomplete_indexName?: string;
  __autocomplete_queryID?: string;
  __autocomplete_algoliaCredentials?: {
    appId: string;
    apiKey: string;
  };
  __autocomplete_id?: number;
}
```

**Key Fields**:

- **objectID**: Unique identifier for the search result
- **url**: Direct link to the documentation page
- **type**: Indicates hierarchy level or content type
- **hierarchy**: Structured navigation path
- **content**: Text excerpt from the page
- **_highlightResult**: Search terms highlighted in results
- **_snippetResult**: Context around search matches

**Example Hit**:
```typescript
{
  objectID: "abc123",
  content: "useState is a React Hook that lets you add state to your component.",
  url: "https://react.dev/reference/react/useState",
  url_without_anchor: "https://react.dev/reference/react/useState",
  type: "lvl1",
  anchor: "usage",
  hierarchy: {
    lvl0: "API Reference",
    lvl1: "useState",
    lvl2: "Usage",
    lvl3: null,
    lvl4: null,
    lvl5: null,
    lvl6: null
  },
  _highlightResult: { /* ... */ },
  _snippetResult: { /* ... */ }
}
```

---

## InternalDocSearchHit Type

Extended search hit with parent relationship.

```typescript { .api }
/**
 * Internal DocSearch hit with parent reference
 * Used for hierarchical navigation
 */
type InternalDocSearchHit = DocSearchHit & {
  /** Parent hit in documentation hierarchy */
  __docsearch_parent: InternalDocSearchHit | null;
};
```

**Usage**: Enables traversing the documentation hierarchy by linking child hits to their parent pages.

```typescript
function getFullPath(hit: InternalDocSearchHit): string[] {
  const path = [hit.hierarchy.lvl1 || hit.hierarchy.lvl0];
  let current = hit.__docsearch_parent;

  while (current) {
    path.unshift(current.hierarchy.lvl1 || current.hierarchy.lvl0);
    current = current.__docsearch_parent;
  }

  return path;
}
```

---

## Dependencies

Required packages for Algolia integration.

```typescript { .api }
/**
 * Required dependencies
 */
import { liteClient, type Hit, type SearchResponse } from 'algoliasearch/lite';
import type { DocSearchHit, InternalDocSearchHit } from '../types/algolia';
```

**Package Version**:
- `algoliasearch@^5.23.3` (lite version for smaller bundle size)

**Lite Client Features**:
- Search-only functionality (no indexing)
- Smaller bundle size
- Optimized for browser and Node.js
- Read-only API key support

---

## Error Handling

The queryAlgolia function handles fetch errors gracefully.

**Fetch Failure Handling**:

```typescript
fetch(hit.url, { headers: { 'User-Agent': '...' } })
  .then(res => {
    if (res.ok) {
      return res.text();
    } else {
      console.error(`Could not fetch docs: ${res.status} ${res.statusText}`);
      return null;
    }
  })
```

**Error Scenarios**:
- HTTP error status (4xx, 5xx): Logs error, returns null
- Network failure: Promise rejects, caught by caller
- Invalid URL: URL constructor throws, caught by caller

**Filtered Results**:
```typescript
return pages.filter(page => page !== null);
```

Null responses (failed fetches) are automatically filtered from results.

---

## User-Agent Header

The queryAlgolia function uses a realistic User-Agent to avoid bot detection.

```typescript
headers: {
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36'
}
```

**Purpose**:
- Mimics a real browser request
- Avoids being blocked by anti-bot measures
- Ensures consistent fetching of documentation pages
