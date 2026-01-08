# React Compiler ESLint Rule Configuration Tool

A command-line tool that helps developers configure ESLint rules for a React project using the React Compiler plugin with advanced error reporting options.

## Capabilities

### Parse command-line configuration options

- Accepts `--severity` flag with comma-separated values: "invalid-react", "invalid-js", "todo" that map to compiler error severity levels [@test](./test/config-parser.test.js)
- Accepts `--bailouts` boolean flag to enable experimental bailout reporting mode [@test](./test/config-parser.test.js)
- Returns null when no flags are provided [@test](./test/config-parser.test.js)

### Generate ESLint configuration object

- Creates a valid ESLint configuration with the plugin "react-compiler" and rule "react-compiler/react-compiler" enabled at "error" level [@test](./test/config-generator.test.js)
- Includes rule options with a Set of severity level enums that filter which compiler errors are reported, based on input severity strings that map to: InvalidReact, InvalidJS, and Todo levels [@test](./test/config-generator.test.js)
- Enables experimental function-level bailout reporting in rule options when bailout flag is true [@test](./test/config-generator.test.js)
- Uses default severity levels (InvalidReact and InvalidJS) when severity list is empty [@test](./test/config-generator.test.js)

## Implementation

[@generates](./src/index.js)

## API

```javascript { #api }
/**
 * Parses command-line arguments to extract React Compiler ESLint configuration options.
 *
 * @param {string[]} args - Array of command-line arguments
 * @returns {Object|null} Configuration object with severity and bailouts properties, or null if no valid options
 */
function parseArgs(args) {
  // IMPLEMENTATION HERE
}

/**
 * Generates an ESLint configuration object for the React Compiler plugin.
 *
 * @param {Object} options - Configuration options
 * @param {string[]} options.severity - Array of severity level strings ("invalid-react", "invalid-js", "todo")
 * @param {boolean} options.bailouts - Whether to enable experimental bailout reporting
 * @returns {Object} ESLint configuration object with plugins, rules, and options
 */
function generateESLintConfig(options) {
  // IMPLEMENTATION HERE
}

module.exports = {
  parseArgs,
  generateESLintConfig,
};
```

## Dependencies { .dependencies }

### eslint-plugin-react-compiler { .dependency }

Provides the React Compiler ESLint plugin with advanced configuration options for error severity filtering and bailout reporting.

[@satisfied-by](eslint-plugin-react-compiler)
