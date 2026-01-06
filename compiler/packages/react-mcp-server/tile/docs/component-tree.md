# React Component Tree Parsing

Parse the component tree from a running React application using Chrome DevTools Protocol. This tool connects to a Chrome browser running in debug mode and extracts the current component tree structure via React DevTools hooks.

## Capabilities

### Parse React Component Tree Tool

Extract the component tree from a running React application by connecting to Chrome DevTools and accessing React's internal component tree representation.

```typescript { .api }
/**
 * Parse React component tree from running application
 *
 * This tool connects to a Chrome browser in debug mode and extracts the
 * React component tree structure from a running application. It uses the
 * Chrome DevTools Protocol and React DevTools internals.
 *
 * Requirements:
 * - Chrome browser running in debug mode on port 9222
 * - React application running at specified URL (default: http://localhost:3000)
 * - React DevTools hook available in the application
 *
 * Setup Chrome Debug Mode:
 * - macOS: "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome --remote-debugging-port=9222 --user-data-dir=/tmp/chrome"
 * - Windows: "chrome.exe --remote-debugging-port=9222 --user-data-dir=C:\\temp\\chrome"
 *
 * @param url - URL of the React application (default: 'http://localhost:3000')
 * @returns Component tree structure as a string representation
 */
interface ParseReactComponentTreeTool {
  name: 'parse-react-component-tree';
  description: 'This tool gets the component tree of a React App. Passing in a url will attempt to connect to the browser and get the current state of the component tree. If no url is passed in, the default url will be used (http://localhost:3000).';
  inputSchema: {
    url?: string; // default: 'http://localhost:3000'
  };
}
```

**Parameters:**

```typescript { .api }
interface ParseReactComponentTreeParams {
  /**
   * URL of the running React application
   *
   * Requirements:
   * - Must include protocol (http:// or https://)
   * - Must include domain (e.g., localhost:3000)
   *
   * Default: 'http://localhost:3000'
   */
  url?: string;
}
```

**Response Format:**

```typescript { .api }
interface ParseComponentTreeResponse {
  content: [{
    type: 'text';
    text: string; // Component tree structure
  }];
  isError?: boolean;
}
```

Successful extraction returns component tree:
```typescript
{
  content: [{
    type: 'text',
    text: '<component tree structure as string>'
  }]
}
```

Error returns:
```typescript
{
  isError: true,
  content: [{
    type: 'text',
    text: 'Error: <error message>'
  }]
}
```

## Setup Requirements

### Chrome Debug Mode

The tool requires Chrome to be running in debug mode with remote debugging enabled on port 9222.

**macOS:**
```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome
```

**Windows:**
```cmd
chrome.exe --remote-debugging-port=9222 --user-data-dir=C:\temp\chrome
```

**Linux:**
```bash
google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome
```

### React Application

The React application must:
1. Be running and accessible at the specified URL
2. Have React DevTools hooks available
3. Be open in a Chrome browser tab (connected to the debug port)

## Usage Examples

### Default URL (localhost:3000)

```typescript
// Parse component tree from default URL
{
  tool: 'parse-react-component-tree',
  params: {} // Uses default: http://localhost:3000
}
```

### Custom URL

```typescript
// Parse component tree from custom URL
{
  tool: 'parse-react-component-tree',
  params: {
    url: 'http://localhost:8080'
  }
}
```

### Production URL

```typescript
// Parse component tree from production URL
{
  tool: 'parse-react-component-tree',
  params: {
    url: 'https://example.com/app'
  }
}
```

## Component Tree Structure

The tool returns a string representation of the React component tree, which typically includes:

- Component names and hierarchy
- Component types (function/class components)
- Component props (potentially)
- Component state (potentially)
- Component hooks (potentially)
- Parent-child relationships

Example output structure:
```
App
├─ Header
│  ├─ Logo
│  └─ Navigation
│     ├─ NavLink
│     └─ NavLink
├─ Main
│  ├─ Sidebar
│  └─ Content
│     ├─ Article
│     └─ Comments
└─ Footer
```

## Error Handling

### Common Errors

**Chrome not running in debug mode:**
```
Error: Failed extract component tree
Could not connect to Chrome on port 9222
```

**Solution:** Start Chrome with `--remote-debugging-port=9222`

**Page not found:**
```
Error: Could not open the page at http://localhost:3000. Is your server running?
```

**Solution:** Verify the React application is running and accessible at the URL

**React DevTools hook not available:**
```
Error: Failed extract component tree
Cannot read property '__REACT_DEVTOOLS_GLOBAL_HOOK__' of undefined
```

**Solution:** Ensure React is loaded and React DevTools hooks are available

## Implementation Details

The tool:
1. Connects to Chrome via Puppeteer using `puppeteer.connect()`
2. Connects to `http://127.0.0.1:9222` (Chrome DevTools Protocol endpoint)
3. Gets all open browser pages
4. Finds the page matching the specified URL
5. Evaluates JavaScript in the page context to access:
   - `window.__REACT_DEVTOOLS_GLOBAL_HOOK__`
   - `rendererInterfaces.get(1)`
   - `__internal_only_getComponentTree()`
6. Returns the component tree as a string

```typescript { .api }
// Internal implementation (not directly accessible)
async function parseReactComponentTree(url: string): Promise<string> {
  const browser = await puppeteer.connect({
    browserURL: 'http://127.0.0.1:9222',
    defaultViewport: null,
  });

  const pages = await browser.pages();
  const targetPage = pages.find(page => page.url().startsWith(url));

  if (!targetPage) {
    throw new Error(`Could not open the page at ${url}. Is your server running?`);
  }

  const componentTree = await targetPage.evaluate(() => {
    return (window as any).__REACT_DEVTOOLS_GLOBAL_HOOK__
      .rendererInterfaces
      .get(1)
      .__internal_only_getComponentTree();
  });

  return componentTree;
}
```

## Use Cases

### Development Debugging

```typescript
// Inspect component hierarchy during development
{
  tool: 'parse-react-component-tree',
  params: {
    url: 'http://localhost:3000'
  }
}

// Use output to understand component structure and relationships
```

### Component Profiling

```typescript
// Analyze component tree before optimization
{
  tool: 'parse-react-component-tree',
  params: {
    url: 'http://localhost:3000/dashboard'
  }
}

// Compare with tree after optimization to verify changes
```

### Architecture Analysis

```typescript
// Understand application component architecture
{
  tool: 'parse-react-component-tree',
  params: {
    url: 'http://localhost:3000'
  }
}

// Use tree to identify component organization patterns
```

## Best Practices

1. **Start Chrome in debug mode first:** Always run Chrome with `--remote-debugging-port=9222` before using the tool
2. **Navigate to the page:** Ensure the target URL is open in a Chrome tab
3. **Wait for React to load:** Let the application fully load before parsing the tree
4. **Use correct URL format:** Include protocol (http:// or https://) and domain
5. **Check for React DevTools:** Verify React DevTools hooks are available (development builds have better support)

## Limitations

1. **Chrome-only:** Tool only works with Chrome browser (not Firefox, Safari, etc.)
2. **Debug mode required:** Chrome must be started with remote debugging enabled
3. **Port 9222:** Tool connects to port 9222 specifically (not configurable)
4. **React DevTools dependency:** Requires React DevTools global hook to be present
5. **Page must be open:** The URL must be open in a Chrome tab
6. **Production builds:** May have limited information in production React builds

## When to Use

- Inspecting React component hierarchy in running applications
- Debugging component structure issues
- Understanding application architecture
- Analyzing component relationships
- Profiling component trees before/after optimizations
- Verifying component composition patterns
- Documenting application component structure
