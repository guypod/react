# Component Tree API

Extract React component tree from running applications using Chrome DevTools Protocol.

## Capabilities

### Parse React Component Tree

Connects to a Chrome browser in debug mode and extracts the React component tree structure from a running application using the React DevTools hook.

```typescript { .api }
/**
 * Extract React component tree from running application
 * @param url - Application URL to extract tree from
 * @returns Promise resolving to component tree as JSON string
 * @throws Error if page not found or extraction fails
 */
function parseReactComponentTree(url: string): Promise<string>;
```

**Requirements**:

1. **Chrome in Debug Mode**: Chrome must be running with remote debugging enabled on port 9222

2. **React Application Running**: The target application must be running at the specified URL

3. **React DevTools Hook**: The application must have the React DevTools global hook available

**Chrome Setup**:

**macOS**:
```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome
```

**Windows**:
```bash
chrome.exe \
  --remote-debugging-port=9222 \
  --user-data-dir=C:\temp\chrome
```

**Linux**:
```bash
google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome
```

**Extraction Process**:

1. **Connect to Browser**: Connect to Puppeteer at `http://127.0.0.1:9222`
   ```typescript
   const browser = await puppeteer.connect({
     browserURL: 'http://127.0.0.1:9222',
     defaultViewport: null
   });
   ```

2. **Find Target Page**: Iterate through all open pages to find matching URL
   ```typescript
   const pages = await browser.pages();
   let localhostPage = null;

   for (const page of pages) {
     const pageUrl = await page.url();
     if (pageUrl.startsWith(url)) {
       localhostPage = page;
       break;
     }
   }
   ```

3. **Extract Component Tree**: Evaluate React DevTools hook in page context
   ```typescript
   const componentTree = await localhostPage.evaluate(() => {
     return window.__REACT_DEVTOOLS_GLOBAL_HOOK__
       .rendererInterfaces.get(1)
       .__internal_only_getComponentTree();
   });
   ```

4. **Return Tree**: Return component tree as string (typically JSON format)

**Usage Example**:

```typescript
import { parseReactComponentTree } from 'react-mcp-server/src/tools/componentTree';

// Default localhost URL
try {
  const tree = await parseReactComponentTree('http://localhost:3000');
  console.log('Component tree:', tree);
} catch (error) {
  console.error('Failed to extract tree:', error.message);
}

// Custom port
const treeCustom = await parseReactComponentTree('http://localhost:8080');

// Custom domain
const treeProd = await parseReactComponentTree('https://myapp.example.com');
```

**Response Format**:

The component tree is returned as a JSON string representing the React component hierarchy. The exact format depends on the React DevTools internal representation, but typically includes:

- Component names
- Component props
- Component state
- Child components
- Fiber node information

**Example Output Structure** (conceptual):
```json
{
  "displayName": "App",
  "type": "function",
  "children": [
    {
      "displayName": "Header",
      "type": "function",
      "props": { "title": "My App" },
      "children": []
    },
    {
      "displayName": "Main",
      "type": "function",
      "children": [
        {
          "displayName": "Sidebar",
          "type": "function",
          "children": []
        },
        {
          "displayName": "Content",
          "type": "function",
          "children": []
        }
      ]
    }
  ]
}
```

---

## Error Handling

The function throws errors in various failure scenarios.

**Error: Page Not Found**

```typescript
throw new Error(
  `Could not open the page at ${url}. Is your server running?`
);
```

**Causes**:
- No page matching the URL is found in open Chrome tabs
- The URL does not exactly match any open page (uses `startsWith` matching)
- Application not running at specified URL

**Error: Failed to Extract Component Tree**

```typescript
throw new Error('Failed extract component tree' + error);
```

**Causes**:
- React DevTools hook not available (`window.__REACT_DEVTOOLS_GLOBAL_HOOK__` is undefined)
- Renderer interface not initialized (`.get(1)` returns undefined)
- `__internal_only_getComponentTree()` method not available or throws error
- Browser connection fails

**Debugging Tips**:

1. **Verify Chrome is Running in Debug Mode**:
   - Open `http://127.0.0.1:9222` in a browser
   - Should see list of inspectable pages

2. **Check React DevTools Hook**:
   - Open Chrome DevTools console on your app
   - Run: `window.__REACT_DEVTOOLS_GLOBAL_HOOK__`
   - Should return an object, not undefined

3. **Verify URL Matching**:
   - Ensure the URL passed to `parseReactComponentTree` matches the beginning of the actual page URL
   - Uses `pageUrl.startsWith(url)` for matching

4. **Check React Version**:
   - React DevTools integration varies by version
   - Ensure React version is compatible with DevTools

---

## React DevTools Integration

The component tree extraction relies on React's internal DevTools global hook.

### DevTools Global Hook

```typescript { .api }
/**
 * React DevTools global hook (browser-side)
 * Available in pages with React DevTools support
 */
interface ReactDevToolsGlobalHook {
  /** Map of renderer IDs to renderer interfaces */
  rendererInterfaces: Map<number, RendererInterface>;

  /** Additional DevTools properties */
  [key: string]: any;
}

/**
 * Renderer interface for React fiber tree access
 */
interface RendererInterface {
  /**
   * Internal-only method to get component tree
   * @returns Component tree structure as JSON-serializable object
   */
  __internal_only_getComponentTree(): any;

  /** Additional renderer methods */
  [key: string]: any;
}
```

**Renderer ID**: The function uses renderer ID `1`, which typically corresponds to the first React root in the application. For apps with multiple React roots, you may need to try different renderer IDs.

**Checking Available Renderers**:
```javascript
// In browser console
const hook = window.__REACT_DEVTOOLS_GLOBAL_HOOK__;
console.log('Available renderers:', Array.from(hook.rendererInterfaces.keys()));
```

---

## URL Matching Behavior

The function uses `startsWith` matching for URL comparison.

**Matching Examples**:

```typescript
// Exact match
parseReactComponentTree('http://localhost:3000')
// Matches: http://localhost:3000

// Path matching
parseReactComponentTree('http://localhost:3000/app')
// Matches: http://localhost:3000/app
// Also matches: http://localhost:3000/app/dashboard

// Query parameters ignored
parseReactComponentTree('http://localhost:3000')
// Matches: http://localhost:3000?debug=true

// Hash fragments ignored
parseReactComponentTree('http://localhost:3000')
// Matches: http://localhost:3000#section-1
```

**Best Practices**:
- Use the base URL without query parameters or hash fragments
- Include the protocol (`http://` or `https://`)
- Include the port if not using default ports (80/443)
- Match the exact URL shown in Chrome's address bar

---

## Dependencies

Required packages for component tree extraction.

```typescript { .api }
/**
 * Required dependency
 */
import puppeteer from 'puppeteer';
```

**Package Version**:
- `puppeteer@^24.7.2`

**Puppeteer Connection**:

The function uses `puppeteer.connect()` instead of `puppeteer.launch()` to connect to an existing Chrome instance:

```typescript
const browser = await puppeteer.connect({
  browserURL: 'http://127.0.0.1:9222',
  defaultViewport: null
});
```

**Key Options**:
- `browserURL`: WebSocket endpoint for Chrome DevTools Protocol
- `defaultViewport: null`: Preserves the browser's existing viewport settings

---

## Use Cases

### Development Debugging

Extract component tree during development to understand application structure:

```typescript
// In development script
const tree = await parseReactComponentTree('http://localhost:3000');
console.log('App structure:', JSON.parse(tree));
```

### Testing

Verify component hierarchy in integration tests:

```typescript
// In test file
const tree = await parseReactComponentTree('http://localhost:3000');
const parsed = JSON.parse(tree);

expect(parsed.displayName).toBe('App');
expect(parsed.children).toHaveLength(3);
```

### Performance Analysis

Analyze component tree depth and complexity:

```typescript
const tree = await parseReactComponentTree('http://localhost:3000');
const parsed = JSON.parse(tree);

function countComponents(node) {
  return 1 + (node.children?.reduce((sum, child) =>
    sum + countComponents(child), 0
  ) || 0);
}

const totalComponents = countComponents(parsed);
console.log('Total components:', totalComponents);
```

### Documentation Generation

Generate component hierarchy documentation:

```typescript
const tree = await parseReactComponentTree('http://localhost:3000');
const parsed = JSON.parse(tree);

function generateDocs(node, depth = 0) {
  const indent = '  '.repeat(depth);
  console.log(`${indent}- ${node.displayName}`);
  node.children?.forEach(child => generateDocs(child, depth + 1));
}

generateDocs(parsed);
```
