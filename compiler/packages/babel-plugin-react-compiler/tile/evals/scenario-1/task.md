# React Compiler Configuration Monitor

A tool that monitors and reports compilation statistics when using a Babel plugin for React optimization.

## Capabilities

### Tracks compilation success events

- When a function is successfully compiled, the monitor logs the function name and file path. [@test](../test/monitor.test.ts)
- When multiple functions are successfully compiled, the monitor logs each function's name separately. [@test](../test/monitor-multiple.test.ts)

### Tracks compilation errors

- When a compilation error occurs, the monitor logs the error kind and file path. [@test](../test/monitor-error.test.ts)

### Configures selective compilation

- The monitor can be configured to only compile files matching a specific pattern. [@test](../test/monitor-selective.test.ts)

### Generates compilation report

- After processing files, the monitor generates a summary report containing total successful compilations and total errors. [@test](../test/monitor-report.test.ts)

## Implementation

[@generates](./src/monitor.ts)

## API

```typescript { #api }
/**
 * Configuration options for the React Compiler Monitor.
 */
export interface MonitorConfig {
  /**
   * Filter function to determine which files should be compiled.
   * Returns true if the file should be compiled, false otherwise.
   */
  fileFilter?: (filename: string) => boolean;

  /**
   * Compilation mode to use.
   * - 'infer': Compile functions that look like components/hooks
   * - 'all': Compile all top-level functions
   */
  compilationMode?: 'infer' | 'all';
}

/**
 * Statistics about compilation activity.
 */
export interface CompilationStats {
  /** Total number of successful compilations */
  successCount: number;

  /** Total number of compilation errors */
  errorCount: number;

  /** List of successfully compiled function names */
  compiledFunctions: string[];

  /** List of file paths with errors */
  errorFiles: string[];
}

/**
 * Creates a React Compiler Monitor with the given configuration.
 *
 * @param config - Configuration options for the monitor
 * @returns An object with methods to get the Babel plugin and retrieve statistics
 */
export function createCompilerMonitor(config?: MonitorConfig): {
  /**
   * Returns the configured Babel plugin array suitable for use in Babel config.
   * Format: [pluginFunction, options]
   */
  getPlugin: () => [any, any];

  /**
   * Returns the current compilation statistics.
   */
  getStats: () => CompilationStats;

  /**
   * Resets the compilation statistics to zero.
   */
  resetStats: () => void;
};
```

## Dependencies { .dependencies }

### babel-plugin-react-compiler { .dependency }

Provides React optimization through automatic memoization.
