# React Forgive

React Forgive (also known as React Analyzer) is a Visual Studio Code extension that provides intelligent Language Server Protocol (LSP) functionality for analyzing React code. It leverages the React Compiler (babel-plugin-react-compiler) to provide real-time feedback on component optimization opportunities, automatic dependency inference for React hooks (particularly useEffect), and code actions for applying compiler-suggested improvements.

## Package Information

- **Package Name**: react-forgive
- **Display Name**: React Analyzer
- **Package Type**: npm (VS Code Extension)
- **Language**: TypeScript
- **Publisher**: Meta
- **VS Code Version Required**: ^1.96.0
- **Repository**: https://github.com/facebook/react (compiler/packages/react-forgive)

## Installation

Install via VS Code Extensions:

```bash
# From VSIX package
code --install-extension react-forgive-0.0.0.vsix
```

Or install dependencies for development:

```bash
cd compiler/packages/react-forgive
yarn install
yarn run compile
```

## Core Features

React Forgive provides the following key capabilities:

1. **Real-time React Code Analysis**: Analyzes React components using the React Compiler
2. **Auto-Dependency Inference**: Automatically infers and displays effect dependencies for React hooks
3. **Code Lenses**: Visual indicators showing functions optimized by the React Compiler
4. **Code Actions**: Quick fixes to apply inferred dependency arrays to React hooks
5. **Visual Decorations**: Highlights inferred effect dependencies in the editor

## Architecture

React Forgive follows a client-server architecture:

- **Client**: VS Code extension host integration (decorations, commands, UI)
- **Server**: LSP server performing code analysis via Babel and React Compiler

The extension activates for JavaScript React (.jsx) and TypeScript React (.tsx) files.

## Basic Usage

Once installed, the extension automatically activates when you open React files (.jsx or .tsx). It will:

1. **Display code lenses** on functions optimized by the React Compiler
2. **Show visual decorations** (bottom borders) on inferred effect dependencies when hovering over useEffect calls
3. **Provide quick fixes** (code actions) to automatically insert inferred dependency arrays

### Toggle Extension

Use the command palette to toggle the extension on/off:

```
Command Palette > React Analyzer: Toggle on/off
```

## Capabilities

### Extension Lifecycle

Extension activation and deactivation functions for integrating with VS Code.

```typescript { .api }
function activate(context: vscode.ExtensionContext): void;
function deactivate(): Thenable<void> | undefined;
```

[Extension Lifecycle](./extension-lifecycle.md)

### Auto-Dependency Decorations

Functionality for requesting and displaying inferred React hook dependencies. This is the core feature that enables automatic dependency inference visualization.

```typescript { .api }
function requestAutoDepsDecorations(
  client: LanguageClient,
  position: vscode.Position,
  options: AutoDepsDecorationsOptions,
): void;

type AutoDepsDecorationsLSPEvent = {
  useEffectCallExpr: [Position, Position];
  decorations: Array<[Position, Position]>;
};
```

[Auto-Dependency Decorations](./auto-deps.md)

### Language Server Protocol Implementation

The LSP server that performs code analysis using Babel and React Compiler. Provides code lenses, code actions, and handles document synchronization.

```typescript { .api }
// Server capabilities
interface ServerCapabilities {
  textDocumentSync: TextDocumentSyncKind.Full;
  codeLensProvider: { resolveProvider: boolean };
  codeActionProvider: { resolveProvider: boolean };
}

// Event handlers
function onInitialize(params: InitializeParams): InitializeResult;
function onCodeLens(params): CodeLens[] | null;
function onCodeAction(params): CodeAction[];
```

[LSP Server](./lsp-server.md)

### Code Compilation

Integration with Babel and React Compiler for analyzing React code. Compiles source code and formats output with Prettier.

```typescript { .api }
function compile(options: CompileOptions): Promise<BabelCore.BabelFileResult | null>;

type CompileOptions = {
  text: string;
  file: string;
  options: PluginOptions | null;
};
```

[Compilation](./compilation.md)

### Utilities

Helper functions for position mapping, range manipulation, and color utilities.

```typescript { .api }
function positionLiteralToVSCodePosition(position: Position): vscode.Position;
function positionsToRange(start: Position, end: Position): vscode.Range;
function isPositionWithinRange(position: Position, range: Range): boolean;
function isRangeWithinRange(aRange: Range, bRange: Range): boolean;
```

[Utilities](./utilities.md)

## VS Code Commands

### react-forgive.toggleAll
**Title**: "React Analyzer: Toggle on/off"
**Description**: User-facing command to enable/disable the extension

### react.requestAutoDepsDecorations
**Type**: Internal command
**Description**: Internal command to request auto-dependency decorations from the LSP server

## Activation Events

The extension activates on the following events:

```json
"activationEvents": [
  "onLanguage:javascriptreact",
  "onLanguage:typescriptreact"
]
```

## Supported Languages

- JavaScript React (javascriptreact)
- TypeScript React (typescriptreact)
- JavaScript (javascript)
- TypeScript (typescript)

## Configuration

### React Compiler Effect Dependency Inference

The extension is preconfigured to infer effect dependencies for the following hooks:

- `React.useEffect` - dependency array at index 1
- `shared-runtime.useSpecialEffect` - dependency array at index 2
- `useEffectWrapper` default export - dependency array at index 1

## Dependencies

### Client Dependencies
- vscode-languageclient ^9.0.1 - LSP client implementation

### Server Dependencies
- @babel/core ^7.26.0 - JavaScript/TypeScript parser and transformer
- @babel/parser ^7.26.0 - Babel parser
- @babel/types ^7.26.0 - Babel AST types
- babel-plugin-react-compiler - React Compiler plugin
- prettier ^3.3.3 - Code formatter
- vscode-languageserver ^9.0.1 - LSP server implementation
- vscode-languageserver-textdocument ^1.0.12 - Text document management

## Visual Features

### Text Editor Decorations
**Type**: inferredEffectDepDecoration
**Styling**:
- Border Color: `diffEditor.move.border` (VS Code theme color)
- Border Style: Solid bottom border (4px)
- Hover Message: "Inferred as an effect dependency"

### Code Lenses
**Display**: "Optimized by React Compiler"
**Location**: On function declarations that were successfully compiled by the React Compiler

### Code Actions
**Type**: Quick fixes
**Purpose**: Automatically apply inferred dependency arrays to React hooks
