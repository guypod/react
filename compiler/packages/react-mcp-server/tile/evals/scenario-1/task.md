# React Compiler MCP Tool

Build a tool that exposes the React Compiler through the Model Context Protocol, allowing AI assistants to compile and analyze React components with pipeline introspection capabilities.

## Capabilities

### Basic Compilation

- Compiles a simple React component with useState and returns the optimized code [@test](../test/compile-basic.test.ts)
- Returns bailout diagnostic messages when code cannot be optimized due to Rules of React violations [@test](../test/compile-bailout.test.ts)

### Pipeline Pass Inspection

- Returns HIR (High-level Intermediate Representation) output when 'HIR' pass is requested [@test](../test/inspect-hir.test.ts)
- Returns ReactiveFunction representation when 'ReactiveFunction' pass is requested [@test](../test/inspect-reactive.test.ts)

### MCP Server Integration

- Creates an MCP server with tool registration that accepts code text and optional pass name [@test](../test/mcp-server.test.ts)

## Implementation

[@generates](./src/index.ts)

## API

```typescript { #api }
/**
 * Compiles React component code using the React Compiler
 * @param code - The React component source code
 * @param options - Compilation options including optional pipeline pass to inspect
 * @returns Compilation result with optimized code and diagnostics
 */
export function compile(
  code: string,
  options?: { passName?: 'HIR' | 'ReactiveFunction' }
): Promise<{ code: string | null; diagnostics: string[]; passOutput?: string }>;

/**
 * Initializes an MCP server that exposes the compile function as a tool
 * @returns McpServer instance ready to accept connections
 */
export function initializeMCPServer(): McpServer;
```

## Dependencies { .dependencies }

### @modelcontextprotocol/sdk { .dependency }

Provides the MCP server framework for creating tools accessible by AI assistants. Used to create server instances, register tools with schemas, and handle stdio transport.

[@satisfied-by](@modelcontextprotocol/sdk)

### babel-plugin-react-compiler { .dependency }

The React Compiler plugin that optimizes React components by automatically adding memoization. Provides compilation functions, intermediate representation types, and printer utilities.

[@satisfied-by](babel-plugin-react-compiler)

### @babel/core { .dependency }

Core Babel functionality for parsing and transforming JavaScript/TypeScript code. Used to parse component source and transform it through the compiler pipeline.

[@satisfied-by](@babel/core)

### zod { .dependency }

Schema validation library for TypeScript. Used to define and validate tool input schemas in the MCP server.

[@satisfied-by](zod)
