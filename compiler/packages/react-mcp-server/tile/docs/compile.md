# React Code Compilation

Compile React code using React Compiler (babel-plugin-react-compiler) to generate automatically memoized components and hooks. The compiler analyzes React code and adds automatic memoization to reduce unnecessary re-renders.

## Capabilities

### Compile Tool

Compile React components and hooks with React Compiler. Returns compiled JavaScript or TypeScript with automatic memoization, and optionally returns intermediate representations from the compiler pipeline.

```typescript { .api }
/**
 * Compile React code with React Compiler
 *
 * This tool compiles React components and hooks, automatically adding memoization
 * to optimize performance. The compiler analyzes the code and applies transformations
 * to reduce unnecessary re-renders.
 *
 * Key Features:
 * - Automatic memoization of components and hooks
 * - Diagnostic messages for compilation bailouts
 * - Access to compiler intermediate representations (HIR, ReactiveFunction)
 * - Integration with React Compiler pipeline
 *
 * @param text - React component or hook code to compile (TypeScript or JavaScript)
 * @param passName - Optional compiler pass to include in output ('HIR' | 'ReactiveFunction' | 'All' | '@DEBUG')
 * @returns Compiled code with automatic memoization and optional compiler diagnostics
 */
interface CompileTool {
  name: 'compile';
  description: 'Compile code with React Compiler. This tool will return the compiled output, which is automatically memoized React components and hooks, written in JavaScript or TypeScript. You can run this tool whenever you want to check if some React code will compile successfully. You can also run this tool every time you make a suggestion to code, to see how it affects the compiled output. If the compiler returns a diagnostic message, you should read the diagnostic message and try to fix the code and run the compiler again to verify. After compiling code successfully, you should run it through the review-react-runtime to verify the compiled code is faster than the original.';
  inputSchema: {
    text: string;
    passName?: 'HIR' | 'ReactiveFunction' | 'All' | '@DEBUG';
  };
}
```

**Parameters:**

```typescript { .api }
interface CompileParams {
  /** React component or hook source code */
  text: string;

  /**
   * Optional compiler pass to include in the output:
   * - 'HIR': High-level Intermediate Representation (last HIR pass before ReactiveFunction)
   * - 'ReactiveFunction': Reactive function representation (last pass of compilation)
   * - 'All': Both HIR and ReactiveFunction passes
   * - '@DEBUG': All compiler passes with names
   */
  passName?: 'HIR' | 'ReactiveFunction' | 'All' | '@DEBUG';
}
```

**Response Format:**

```typescript { .api }
interface CompileResponse {
  content: Array<{
    type: 'text';
    text: string; // Compiled code or compiler pass output
  }>;
  isError?: boolean;
}
```

Successful compilation returns:
```typescript
{
  content: [
    { type: 'text', text: '<compiled code with automatic memoization>' },
    // Optional: compiler pass outputs if passName specified
    { type: 'text', text: '<HIR or ReactiveFunction representation>' }
  ]
}
```

Compilation with bailouts returns:
```typescript
{
  content: [
    {
      type: 'text',
      text: 'React Compiler bailed out:\n\n<diagnostic message>@<start_line>:<end_line>'
    }
  ]
}
```

Compilation error returns:
```typescript
{
  isError: true,
  content: [{ type: 'text', text: 'Error: <error message>' }]
}
```

## Compiler Output

### Successful Compilation

Successfully compiled code includes:
1. Import statement: `import { c as _c } from "react/compiler-runtime";`
2. Cache initialization: `const $ = _c(n);` where `n` is the cache size (number of memoized values)
3. Memoized code with cache access patterns

Example:
```typescript
// Original code
function Counter({ initial = 0 }) {
  const [count, setCount] = useState(initial);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// Compiled code (simplified)
import { c as _c } from "react/compiler-runtime";
function Counter({ initial = 0 }) {
  const $ = _c(5);
  // Compiler-generated memoization logic
  // ...
}
```

### Compilation Bailouts

When the compiler cannot optimize code, it returns diagnostic messages indicating:
- The reason for bailout
- The location in the code (line numbers)
- Suggestions for fixing the issue

Common bailouts include:
- Existing manual memoization that cannot be preserved (useMemo, useCallback, React.memo)
- Rules of React violations
- Unsupported code patterns

## Usage Examples

### Basic Compilation

```typescript
// Compile a simple component
{
  tool: 'compile',
  params: {
    text: `
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}
    `
  }
}

// Returns compiled code with automatic memoization
```

### Compilation with HIR Pass

```typescript
// Compile and include HIR intermediate representation
{
  tool: 'compile',
  params: {
    text: `
function TodoList({ todos }) {
  const [filter, setFilter] = useState('all');
  const filtered = todos.filter(t =>
    filter === 'all' || t.status === filter
  );
  return <div>{filtered.map(t => <Todo key={t.id} todo={t} />)}</div>;
}
    `,
    passName: 'HIR'
  }
}

// Returns compiled code + HIR representation
```

### Compilation with All Passes

```typescript
// Get compiled code and all intermediate representations
{
  tool: 'compile',
  params: {
    text: `
function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  return { count, increment, decrement };
}
    `,
    passName: 'All'
  }
}

// Returns compiled code + HIR + ReactiveFunction representations
```

### Debugging Compilation

```typescript
// Debug mode with all compiler passes
{
  tool: 'compile',
  params: {
    text: `<component code>`,
    passName: '@DEBUG'
  }
}

// Returns compiled code + all compiler passes with names for debugging
```

## Handling Bailouts

### Manual Memoization Bailout

When the compiler encounters existing manual memoization:

```typescript
// This will bail out
function MyComponent() {
  const value = useMemo(() => expensiveComputation(), []);
  return <div>{value}</div>;
}
```

**Solution:** Remove the manual memoization and recompile:
```typescript
// Fixed version
function MyComponent() {
  const value = expensiveComputation();
  return <div>{value}</div>;
}
```

### Rules of React Violations

The compiler bails out when code violates Rules of React:

```typescript
// This will bail out - conditional hook call
function MyComponent({ condition }) {
  if (condition) {
    useState(0); // ❌ Hooks must be called unconditionally
  }
  return <div>Hello</div>;
}
```

**Solution:** Restructure to follow Rules of React:
```typescript
// Fixed version
function MyComponent({ condition }) {
  const [state] = useState(0); // ✅ Hook called unconditionally
  if (condition) {
    // Use the state conditionally instead
  }
  return <div>Hello</div>;
}
```

## Compilation Options

The compiler uses the following configuration:

```typescript { .api }
interface PluginOptions {
  panicThreshold: 'none';
  logger: {
    debugLogIRs: (result: CompilerPipelineValue) => void;
    logEvent: (filename: string, event: CompilerEvent) => void;
  };
}
```

## Best Practices

1. **Always verify compilation:** Run the compile tool to check if code optimizes successfully
2. **Remove manual memoization:** Let React Compiler handle memoization automatically
3. **Follow Rules of React:** Ensure code follows React's rules for successful compilation
4. **Use review-react-runtime:** After successful compilation, verify performance improvements
5. **Iterate on bailouts:** Read diagnostic messages, fix issues, and recompile
6. **Check compiler output:** Look for `const $ = _c(n)` to verify successful memoization

## When to Use

- Checking if React code will compile successfully
- Optimizing React components and hooks automatically
- Verifying code changes don't break compilation
- Understanding how React Compiler optimizes code
- Debugging compilation issues with intermediate representations
- Removing manual memoization (useMemo, useCallback, React.memo)
