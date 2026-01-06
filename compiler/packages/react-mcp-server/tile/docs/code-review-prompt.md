# React Code Review Prompt

System prompt providing comprehensive guidelines for reviewing and optimizing React code with React Compiler. This prompt configures AI assistants to follow React best practices and leverage React Compiler optimizations.

## Capabilities

### Review React Code Prompt

MCP prompt that returns a detailed system message with guidelines for reviewing React code, optimizing for React Compiler, and following React best practices.

```typescript { .api }
/**
 * React code review and optimization system prompt
 *
 * This prompt provides:
 * - React best practices and coding guidelines
 * - React Compiler optimization strategies
 * - Understanding compiler output and bailouts
 * - Performance optimization techniques
 * - Process for reviewing and improving React code
 *
 * The prompt configures AI assistants to help users write efficient,
 * optimizable React code that leverages React Compiler's automatic
 * memoization capabilities.
 */
interface ReviewReactCodePrompt {
  name: 'review-react-code';
  description: 'System prompt for reviewing and optimizing React code';
  messages: [{
    role: 'assistant';
    content: {
      type: 'text';
      text: string; // Complete guidelines document
    };
  }];
}
```

## Prompt Contents

The prompt provides comprehensive guidance organized into the following sections:

### Role Definition

```typescript { .api }
/**
 * Defines the assistant's role as a React optimization specialist
 *
 * Focus areas:
 * - Writing efficient and optimizable React code
 * - Identifying patterns that enable React Compiler optimization
 * - Reducing unnecessary re-renders
 * - Improving application performance
 */
```

### React Coding Guidelines

The prompt includes detailed guidelines covering:

**Functional Components and Hooks:**
- Use functional components with Hooks (not class components)
- Manage state with useState/useReducer
- Handle side effects with useEffect
- Follow React Hooks rules unconditionally

**Pure Components:**
- Keep rendering pure and side-effect-free
- No side effects in component body
- Wrap side effects in useEffect or event handlers
- Render logic as pure function of props and state

**One-Way Data Flow:**
- Pass data down through props
- Avoid global mutations
- Lift state up to common parents
- Use React Context when needed

**Immutable State Updates:**
- Never mutate state directly
- Use spread syntax for updates
- Use state setters (setState) exclusively
- Create new objects/arrays for updates

**Effect Hook Best Practices:**
- Avoid useEffect when possible (prefer event handlers)
- Never setState within useEffect (degrades performance)
- Include all dependencies in dependency array
- Return cleanup functions
- Use effects only for synchronization with external state

**Rules of Hooks:**
- Call Hooks at top level unconditionally
- No Hooks in loops, conditions, or nested functions
- Only call Hooks from React components or custom Hooks

**Ref Usage:**
- Use refs sparingly (focus, animation, non-React integration)
- Don't use refs for reactive state
- Never read/write ref.current during rendering
- Refs should not affect rendered output

**Component Composition:**
- Break UI into small, reusable components
- Promote clarity and reusability through composition
- Abstract repetitive logic into custom Hooks
- Prefer composition over large monolithic components

**Concurrency Optimization:**
- Write components that handle multiple renders
- Use functional state updates: `setState(prev => prev + 1)`
- Include cleanup functions in effects
- Avoid side effects for "do this when X changes" patterns
- Support React's concurrent rendering

**Network Optimization:**
- Use parallel data fetching (start requests together)
- Leverage Suspense for data loading
- Co-locate requests with components needing the data
- Fetch related data together server-side (Server Components)
- Use caching to avoid duplicate requests

**React Compiler Reliance:**
- Omit manual memoization (useMemo, useCallback, React.memo)
- Let React Compiler handle memoization automatically
- Write clear, simple components with direct data flow
- Focus on side-effect-free render functions
- Trust compiler for tree-shaking and inlining

**User Experience Design:**
- Provide clear, minimal, non-blocking UI states
- Show lightweight placeholders during loading (skeletons)
- Handle errors gracefully (error boundaries or inline messages)
- Render partial data as available (don't wait for everything)
- Use Suspense for natural loading state declarations

**Server Components:**
- Shift data-heavy logic to server
- Break static parts into Server Components
- Break data fetching into Server Components
- Use 'use client' directive only for interactive components
- Pre-render data server-side for faster loads
- Reduce client-side JavaScript bundle size

### Available Tools Reference

```typescript { .api }
/**
 * Tools available for React code analysis
 */
interface AvailableTools {
  /** Look up React.dev documentation */
  docs: 'docs://{query}';

  /** Run React Compiler on code */
  compile: 'Returns optimized JS/TS with diagnostics';
}
```

### Review Process

The prompt defines a structured process:

1. **Analyze Code:**
   - Check for React anti-patterns preventing compiler optimization
   - Identify unnecessary manual optimizations
   - Look for component structure issues limiting compiler effectiveness
   - Consult React docs using docs://{query} resource

2. **Verify with React Compiler:**
   - Run code through compiler and analyze output
   - Run multiple times to verify work
   - Check for successful optimization (`const $ = _c(n)` cache entries)
   - Identify bailout messages indicating improvement opportunities
   - Compare before/after optimization potential

3. **Provide Actionable Guidance:**
   - Explain specific code changes with clear reasoning
   - Show before/after examples when suggesting changes
   - Include compiler results to demonstrate optimization impact
   - Only suggest meaningful optimization improvements

### Optimization Guidelines

```typescript { .api }
/**
 * Key optimization principles for React Compiler
 */
interface OptimizationGuidelines {
  /** Avoid mutation of compiler-memoized values */
  avoidMutation: true;

  /** Structure state updates for granular updates */
  granularUpdates: true;

  /** Isolate side effects with clear dependencies */
  isolateSideEffects: true;

  /** Remove manual memoization (compiler handles it) */
  removeManualMemo: true;
}
```

### Understanding Compiler Output

The prompt explains how to interpret compiler results:

**Successful Optimization Indicators:**
```typescript
// Import of compiler runtime
import { c as _c } from "react/compiler-runtime";

// Cache initialization with size n
const $ = _c(n);
```

**Cache Size Analysis:**
- **Increase cache size (n):** More memoization coverage
- **Decrease cache size (n):** Fewer dependencies, less re-rendering

## Usage Examples

### Using the Prompt

```typescript
// Request the React code review prompt
{
  prompt: 'review-react-code'
}

// Returns system message with complete guidelines
```

### Example Review Workflow

1. **User provides code for review**
2. **Assistant uses review-react-code prompt** to load guidelines
3. **Assistant analyzes code** against guidelines
4. **Assistant uses compile tool** to verify optimization
5. **Assistant provides recommendations** with before/after examples
6. **Assistant uses compile tool again** to verify improvements

### Example Response Structure

```markdown
I'll review your React code for optimization opportunities.

## Analysis

Your code has the following issues:
- Manual memoization with useMemo that React Compiler can handle
- Side effect in useEffect that could be in event handler
- Mutation of state object

## Recommendations

### 1. Remove Manual Memoization

Before:
\`\`\`typescript
const filtered = useMemo(() =>
  items.filter(item => item.active),
  [items]
);
\`\`\`

After:
\`\`\`typescript
const filtered = items.filter(item => item.active);
\`\`\`

Let me verify this compiles successfully...

[Uses compile tool to verify]

### 2. Move Side Effect to Event Handler

Before:
\`\`\`typescript
useEffect(() => {
  logAnalytics('page-view');
}, []);
\`\`\`

After:
\`\`\`typescript
// In component mount or button click handler
const handlePageLoad = () => {
  logAnalytics('page-view');
};
\`\`\`

## Compiler Results

Original code: Failed to compile (bailout: manual memoization)
Optimized code: Successfully compiled with const $ = _c(5)

The cache size of 5 indicates the compiler is memoizing 5 values automatically.
```

## Key Concepts

### Optimization Strategies

```typescript { .api }
/**
 * Strategies for optimizing React code
 */
interface OptimizationStrategies {
  /** Avoid mutation of values memoized by compiler */
  immutability: 'Use spread syntax and immutable updates';

  /** Structure state for granular updates */
  stateStructure: 'Split state into atomic pieces when possible';

  /** Isolate side effects clearly */
  sideEffects: 'Use useEffect only for external synchronization';

  /** Remove manual memoization */
  autoMemoization: 'Let React Compiler handle memoization';

  /** Follow Rules of React */
  rulesOfReact: 'Hooks at top level, pure render functions';
}
```

### Compiler Output Indicators

```typescript { .api }
/**
 * Understanding what compiler output tells you
 */
interface CompilerOutputIndicators {
  /** Successful optimization marker */
  successMarker: 'const $ = _c(n)';

  /** Cache size interpretation */
  cacheSize: {
    meaning: 'Number of memoized values';
    increase: 'More memoization coverage (more expressions cached)';
    decrease: 'Fewer dependencies (less re-rendering needed)';
  };

  /** Import statement */
  runtimeImport: 'import { c as _c } from "react/compiler-runtime"';
}
```

## Best Practices

1. **Use the prompt consistently:** Apply review-react-code prompt when reviewing React code
2. **Verify with compile tool:** Always run compiler to validate optimization suggestions
3. **Show before/after:** Provide clear examples demonstrating improvements
4. **Explain reasoning:** Include rationale for each optimization suggestion
5. **Check compiler output:** Verify successful compilation and memoization
6. **Iterate on bailouts:** Address bailout messages and recompile
7. **Focus on meaningful changes:** Only suggest optimizations that improve performance

## When to Use

- Reviewing React code for optimization opportunities
- Analyzing code for React Compiler compatibility
- Identifying anti-patterns preventing compiler optimization
- Teaching React best practices
- Optimizing React components for performance
- Understanding React Compiler output
- Improving code structure for automatic memoization
