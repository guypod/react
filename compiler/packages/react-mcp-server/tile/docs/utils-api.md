# Utilities API

Helper functions and utilities used throughout the React MCP Server.

## Capabilities

### Assert Exhaustive

TypeScript exhaustiveness check that throws at runtime for unhandled union cases.

```typescript { .api }
/**
 * Trigger an exhaustiveness check in TypeScript and throw at runtime
 * @param _ - Value that should be never type (exhaustiveness check)
 * @param errorMsg - Error message to throw if this code is reached
 * @returns never - Function always throws, never returns
 * @throws Error with the provided error message
 */
function assertExhaustive(_: never, errorMsg: string): never;
```

**Purpose**: Ensures all cases in a union type are handled in switch statements or conditional logic. Provides compile-time type safety and runtime error reporting.

**TypeScript Benefits**:
- Compile-time error if not all union cases are handled
- Type narrowing ensures the `_` parameter is `never` type
- Helps catch bugs when adding new union variants

**Runtime Behavior**:
- Always throws an Error with the provided message
- Should never be reached if TypeScript checks are satisfied
- Useful for catching logic errors in production

**Usage Example**:

```typescript
import assertExhaustive from 'react-mcp-server/src/utils/assertExhaustive';

type CompilerPass = 'HIR' | 'ReactiveFunction' | 'All' | '@DEBUG';

function handlePass(pass: CompilerPass): void {
  switch (pass) {
    case 'HIR':
      console.log('Handling HIR pass');
      break;
    case 'ReactiveFunction':
      console.log('Handling ReactiveFunction pass');
      break;
    case 'All':
      console.log('Handling All passes');
      break;
    case '@DEBUG':
      console.log('Handling DEBUG pass');
      break;
    default:
      // TypeScript ensures this is never reached
      assertExhaustive(pass, `Unhandled pass type: ${pass}`);
  }
}

// If a new pass type is added to CompilerPass, TypeScript will error
// at the assertExhaustive call, forcing you to handle the new case
```

**Type Safety**:

```typescript
type Status = 'pending' | 'success' | 'error';

function processStatus(status: Status): string {
  if (status === 'pending') {
    return 'Processing...';
  } else if (status === 'success') {
    return 'Complete!';
  }
  // TypeScript error: status is 'error', not never
  // Must handle 'error' case before assertExhaustive
  assertExhaustive(status, `Unhandled status: ${status}`);
}
```

**Adding New Union Cases**:

```typescript
// Original type
type Action = 'start' | 'stop';

function handleAction(action: Action): void {
  switch (action) {
    case 'start':
      console.log('Starting');
      break;
    case 'stop':
      console.log('Stopping');
      break;
    default:
      assertExhaustive(action, `Unknown action: ${action}`);
  }
}

// Add new action type
type Action = 'start' | 'stop' | 'pause';

// TypeScript now errors at assertExhaustive because 'pause' is not handled
// This forces you to add a case for 'pause', ensuring completeness
```

**Error Message Format**:

```typescript
// Good: Include the unhandled value in the message
assertExhaustive(value, `Unhandled case: ${value}`);

// Better: Include context about where the error occurred
assertExhaustive(
  passName,
  `Unhandled passName option in compile tool: ${passName}`
);
```

---

## Usage Patterns

### Exhaustiveness Checking in Switch Statements

```typescript
import assertExhaustive from 'react-mcp-server/src/utils/assertExhaustive';

type Result = { kind: 'hir'; value: string }
  | { kind: 'reactive'; value: string }
  | { kind: 'debug'; value: string };

function processResult(result: Result): void {
  switch (result.kind) {
    case 'hir':
      console.log('HIR:', result.value);
      break;
    case 'reactive':
      console.log('Reactive:', result.value);
      break;
    case 'debug':
      console.log('Debug:', result.value);
      break;
    default:
      assertExhaustive(result, `Unhandled result type`);
  }
}
```

### Exhaustiveness Checking in If-Else Chains

```typescript
type Mode = 'development' | 'production' | 'test';

function configureEnvironment(mode: Mode): void {
  if (mode === 'development') {
    console.log('Dev mode');
  } else if (mode === 'production') {
    console.log('Prod mode');
  } else if (mode === 'test') {
    console.log('Test mode');
  } else {
    assertExhaustive(mode, `Unknown mode: ${mode}`);
  }
}
```

---

## Best Practices

### Exhaustiveness Checking

1. **Always use in discriminated unions**: Ensures compile-time safety when handling union types

2. **Include descriptive error messages**: Help debugging if the error is ever thrown

3. **Place in default case**: Put assertExhaustive in the default case of switch statements

4. **Use with tagged unions**: Most effective with discriminated unions (types with a common discriminant property)

```typescript
// Good: Tagged union with kind property
type Action =
  | { kind: 'fetch'; url: string }
  | { kind: 'update'; data: any };

function handle(action: Action) {
  switch (action.kind) {
    case 'fetch':
      return fetch(action.url);
    case 'update':
      return update(action.data);
    default:
      assertExhaustive(action, `Unknown action`);
  }
}
```

