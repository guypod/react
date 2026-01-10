# Logging & Events

Logger interface for tracking compilation events, errors, diagnostics, and metrics throughout the compilation process.

## Capabilities

### Logger Interface

Logger interface for compilation events.

```typescript { .api }
/**
 * Logger interface for tracking compilation events
 */
interface Logger {
  /**
   * Log a compilation event
   * @param filename - Source filename (null if unknown)
   * @param event - Event to log
   */
  logEvent(filename: string | null, event: LoggerEvent): void;
}
```

**Usage Example:**

```typescript
import { type Logger, type LoggerEvent } from "babel-plugin-react-compiler";

const logger: Logger = {
  logEvent(filename, event) {
    const timestamp = new Date().toISOString();
    const file = filename || "unknown";

    switch (event.kind) {
      case "CompileSuccess":
        console.log(
          `[${timestamp}] ✓ ${file}: Compiled ${event.fnName || "anonymous"}`
        );
        console.log(`  Memo slots: ${event.memoSlots}`);
        console.log(`  Memo blocks: ${event.memoBlocks}`);
        break;

      case "CompileError":
        console.error(`[${timestamp}] ✗ ${file}: ${event.detail.reason}`);
        break;

      case "CompileDiagnostic":
        console.warn(`[${timestamp}] ⚠ ${file}: ${event.detail.reason}`);
        break;

      case "PipelineError":
        console.error(`[${timestamp}] ⚠ ${file}: Pipeline error`);
        console.error(event.data);
        break;
    }
  },
};

// Use with plugin options
const options = {
  logger,
  compilationMode: "infer",
};
```

## Types

### LoggerEvent

Union type of all compilation events.

```typescript { .api }
/**
 * Compilation events that can be logged
 */
type LoggerEvent =
  | CompileErrorEvent
  | CompileDiagnosticEvent
  | CompileSuccessEvent
  | PipelineErrorEvent;
```

### CompileErrorEvent

Event for compilation errors.

```typescript { .api }
/**
 * Compilation error event
 */
interface CompileErrorEvent {
  /**
   * Event kind
   */
  kind: "CompileError";

  /**
   * Function location where error occurred
   * Null if error not associated with a function
   */
  fnLoc: t.SourceLocation | null;

  /**
   * Error detail
   */
  detail: CompilerErrorDetailOptions;
}
```

### CompileDiagnosticEvent

Event for non-error diagnostics (warnings, info).

```typescript { .api }
/**
 * Diagnostic event (warnings, info)
 */
interface CompileDiagnosticEvent {
  /**
   * Event kind
   */
  kind: "CompileDiagnostic";

  /**
   * Function location where diagnostic occurred
   */
  fnLoc: t.SourceLocation | null;

  /**
   * Diagnostic detail (no severity or suggestions)
   */
  detail: Omit<CompilerErrorDetailOptions, "severity" | "suggestions">;
}
```

### CompileSuccessEvent

Event for successful compilation.

```typescript { .api }
/**
 * Successful compilation event with metrics
 */
interface CompileSuccessEvent {
  /**
   * Event kind
   */
  kind: "CompileSuccess";

  /**
   * Function location
   */
  fnLoc: t.SourceLocation | null;

  /**
   * Function name (null for anonymous)
   */
  fnName: string | null;

  /**
   * Number of memo cache slots used
   * Total $[0], $[1], etc. slots
   */
  memoSlots: number;

  /**
   * Number of memo blocks (reactive scopes)
   */
  memoBlocks: number;

  /**
   * Number of memoized values
   */
  memoValues: number;

  /**
   * Number of pruned memo blocks
   * Scopes optimized away (e.g., for hooks)
   */
  prunedMemoBlocks: number;

  /**
   * Number of values in pruned blocks
   */
  prunedMemoValues: number;
}
```

### PipelineErrorEvent

Event for pipeline-level errors.

```typescript { .api }
/**
 * Pipeline error event
 */
interface PipelineErrorEvent {
  /**
   * Event kind
   */
  kind: "PipelineError";

  /**
   * Function location where pipeline error occurred
   */
  fnLoc: t.SourceLocation | null;

  /**
   * Error data (string representation)
   */
  data: string;
}
```

## Logger Implementation Patterns

### Console Logger

Simple console-based logger:

```typescript
import { type Logger } from "babel-plugin-react-compiler";

const consoleLogger: Logger = {
  logEvent(filename, event) {
    const file = filename || "<unknown>";

    if (event.kind === "CompileSuccess") {
      console.log(`✓ Compiled ${file}:${event.fnName || "anonymous"}`);
    } else if (event.kind === "CompileError") {
      console.error(
        `✗ Error in ${file}: ${event.detail.reason}`
      );
    } else if (event.kind === "CompileDiagnostic") {
      console.warn(
        `⚠ Warning in ${file}: ${event.detail.reason}`
      );
    }
  },
};
```

### File Logger

Logger that writes to a file:

```typescript
import { type Logger } from "babel-plugin-react-compiler";
import { appendFileSync } from "fs";

function createFileLogger(logPath: string): Logger {
  return {
    logEvent(filename, event) {
      const timestamp = new Date().toISOString();
      const entry = {
        timestamp,
        filename: filename || null,
        event,
      };

      appendFileSync(logPath, JSON.stringify(entry) + "\n");
    },
  };
}

const logger = createFileLogger("./compiler.log");
```

### Metrics Collector

Logger that collects compilation metrics:

```typescript
import { type Logger, type LoggerEvent } from "babel-plugin-react-compiler";

class MetricsLogger implements Logger {
  private metrics = {
    totalCompilations: 0,
    successfulCompilations: 0,
    failedCompilations: 0,
    totalMemoSlots: 0,
    totalMemoBlocks: 0,
    totalMemoValues: 0,
    errors: [] as Array<{ filename: string; reason: string }>,
  };

  logEvent(filename: string | null, event: LoggerEvent): void {
    const file = filename || "<unknown>";

    if (event.kind === "CompileSuccess") {
      this.metrics.totalCompilations++;
      this.metrics.successfulCompilations++;
      this.metrics.totalMemoSlots += event.memoSlots;
      this.metrics.totalMemoBlocks += event.memoBlocks;
      this.metrics.totalMemoValues += event.memoValues;
    } else if (event.kind === "CompileError") {
      this.metrics.totalCompilations++;
      this.metrics.failedCompilations++;
      this.metrics.errors.push({
        filename: file,
        reason: event.detail.reason,
      });
    }
  }

  getMetrics() {
    return {
      ...this.metrics,
      successRate:
        this.metrics.totalCompilations > 0
          ? (this.metrics.successfulCompilations / this.metrics.totalCompilations) * 100
          : 0,
      avgMemoSlotsPerCompilation:
        this.metrics.successfulCompilations > 0
          ? this.metrics.totalMemoSlots / this.metrics.successfulCompilations
          : 0,
    };
  }

  printSummary() {
    const metrics = this.getMetrics();

    console.log("\n=== Compilation Metrics ===");
    console.log(`Total compilations: ${metrics.totalCompilations}`);
    console.log(`Successful: ${metrics.successfulCompilations}`);
    console.log(`Failed: ${metrics.failedCompilations}`);
    console.log(`Success rate: ${metrics.successRate.toFixed(1)}%`);
    console.log(`\nMemoization:`);
    console.log(`  Total memo slots: ${metrics.totalMemoSlots}`);
    console.log(`  Total memo blocks: ${metrics.totalMemoBlocks}`);
    console.log(`  Total memoized values: ${metrics.totalMemoValues}`);
    console.log(
      `  Avg slots per compilation: ${metrics.avgMemoSlotsPerCompilation.toFixed(1)}`
    );

    if (metrics.errors.length > 0) {
      console.log(`\nErrors:`);
      metrics.errors.forEach((error, i) => {
        console.log(`  ${i + 1}. ${error.filename}: ${error.reason}`);
      });
    }
  }
}

// Usage
const metricsLogger = new MetricsLogger();

// ... compile files ...

metricsLogger.printSummary();
```

### Filtered Logger

Logger that filters events by type or severity:

```typescript
import { type Logger, type LoggerEvent, ErrorSeverity } from "babel-plugin-react-compiler";

function createFilteredLogger(
  baseLogger: Logger,
  options: {
    includeSuccess?: boolean;
    includeErrors?: boolean;
    includeDiagnostics?: boolean;
    minSeverity?: ErrorSeverity;
  }
): Logger {
  const {
    includeSuccess = true,
    includeErrors = true,
    includeDiagnostics = true,
    minSeverity = ErrorSeverity.Todo,
  } = options;

  return {
    logEvent(filename, event) {
      let shouldLog = false;

      if (event.kind === "CompileSuccess" && includeSuccess) {
        shouldLog = true;
      } else if (event.kind === "CompileError" && includeErrors) {
        // Filter by severity
        const severity = event.detail.severity;
        shouldLog = severity >= minSeverity;
      } else if (event.kind === "CompileDiagnostic" && includeDiagnostics) {
        shouldLog = true;
      } else if (event.kind === "PipelineError") {
        shouldLog = true;
      }

      if (shouldLog) {
        baseLogger.logEvent(filename, event);
      }
    },
  };
}

// Usage - only log errors and critical issues
const filteredLogger = createFilteredLogger(consoleLogger, {
  includeSuccess: false,
  includeDiagnostics: false,
  includeErrors: true,
  minSeverity: ErrorSeverity.InvalidReact,
});
```

### Multi-destination Logger

Logger that forwards to multiple loggers:

```typescript
import { type Logger, type LoggerEvent } from "babel-plugin-react-compiler";

function createMultiLogger(...loggers: Logger[]): Logger {
  return {
    logEvent(filename, event) {
      for (const logger of loggers) {
        logger.logEvent(filename, event);
      }
    },
  };
}

// Usage - log to both console and file
const multiLogger = createMultiLogger(
  consoleLogger,
  createFileLogger("./compiler.log"),
  metricsLogger
);
```

## Event-specific Handlers

### Handle Compilation Success

```typescript
import { type CompileSuccessEvent } from "babel-plugin-react-compiler";

function handleCompileSuccess(event: CompileSuccessEvent, filename: string) {
  console.log(`\n✓ Successfully compiled: ${filename}`);

  if (event.fnName) {
    console.log(`  Function: ${event.fnName}`);
  }

  if (event.fnLoc) {
    console.log(
      `  Location: ${event.fnLoc.start.line}:${event.fnLoc.start.column}`
    );
  }

  console.log(`\n  Optimization metrics:`);
  console.log(`    Memo slots: ${event.memoSlots}`);
  console.log(`    Memo blocks: ${event.memoBlocks}`);
  console.log(`    Memoized values: ${event.memoValues}`);

  if (event.prunedMemoBlocks > 0) {
    console.log(
      `    Pruned blocks: ${event.prunedMemoBlocks} (optimized away)`
    );
    console.log(`    Pruned values: ${event.prunedMemoValues}`);
  }

  // Calculate optimization efficiency
  const totalBlocks = event.memoBlocks + event.prunedMemoBlocks;
  const efficiency =
    totalBlocks > 0 ? (event.memoBlocks / totalBlocks) * 100 : 0;

  console.log(`    Efficiency: ${efficiency.toFixed(1)}%`);
}
```

### Handle Compilation Error

```typescript
import { type CompileErrorEvent } from "babel-plugin-react-compiler";

function handleCompileError(event: CompileErrorEvent, filename: string) {
  console.error(`\n✗ Compilation error: ${filename}`);

  if (event.fnLoc) {
    console.error(
      `  at ${event.fnLoc.start.line}:${event.fnLoc.start.column}`
    );
  }

  console.error(`  Severity: ${event.detail.severity}`);
  console.error(`  Reason: ${event.detail.reason}`);

  if (event.detail.description) {
    console.error(`  Description: ${event.detail.description}`);
  }

  if (event.detail.loc) {
    console.error(
      `  Error location: ${event.detail.loc.start.line}:${event.detail.loc.start.column}`
    );
  }

  if (event.detail.suggestions && event.detail.suggestions.length > 0) {
    console.error(`\n  Suggestions:`);
    event.detail.suggestions.forEach((suggestion, i) => {
      console.error(`    ${i + 1}. ${suggestion.description}`);
    });
  }
}
```

## Integration with Build Tools

### Webpack Plugin Logger

```typescript
import { type Logger } from "babel-plugin-react-compiler";
import type { Compiler } from "webpack";

function createWebpackLogger(compiler: Compiler): Logger {
  return {
    logEvent(filename, event) {
      const file = filename || "<unknown>";

      if (event.kind === "CompileError") {
        compiler.hooks.compilation.tap("ReactCompiler", (compilation) => {
          compilation.errors.push(
            new Error(`[React Compiler] ${file}: ${event.detail.reason}`)
          );
        });
      } else if (event.kind === "CompileSuccess") {
        if (compiler.options.stats?.verbose) {
          console.log(
            `[React Compiler] Optimized ${file}: ${event.memoBlocks} scopes`
          );
        }
      }
    },
  };
}
```

### Vite Plugin Logger

```typescript
import { type Logger } from "babel-plugin-react-compiler";
import type { Logger as ViteLogger } from "vite";

function createViteLogger(viteLogger: ViteLogger): Logger {
  return {
    logEvent(filename, event) {
      const file = filename || "<unknown>";

      if (event.kind === "CompileSuccess") {
        viteLogger.info(
          `[react-compiler] ✓ ${file}: ${event.memoBlocks} scopes, ${event.memoSlots} slots`
        );
      } else if (event.kind === "CompileError") {
        viteLogger.error(
          `[react-compiler] ✗ ${file}: ${event.detail.reason}`
        );
      } else if (event.kind === "CompileDiagnostic") {
        viteLogger.warn(
          `[react-compiler] ⚠ ${file}: ${event.detail.reason}`
        );
      }
    },
  };
}
```

## Best Practices

### Development Logging

For development, use verbose logging with metrics:

```typescript
const devLogger: Logger = {
  logEvent(filename, event) {
    if (event.kind === "CompileSuccess") {
      console.log(`✓ ${filename}: ${event.fnName}`);
      console.log(
        `  ${event.memoBlocks} scopes, ${event.memoSlots} slots, ${event.memoValues} values`
      );
    } else if (event.kind === "CompileError") {
      console.error(`✗ ${filename}: ${event.detail.reason}`);
      if (event.detail.description) {
        console.error(`  ${event.detail.description}`);
      }
    }
  },
};
```

### Production Logging

For production, log only errors and critical metrics:

```typescript
const prodLogger: Logger = {
  logEvent(filename, event) {
    if (event.kind === "CompileError") {
      // Send to error tracking service
      errorTracker.captureError({
        message: event.detail.reason,
        filename,
        severity: event.detail.severity,
      });
    }
  },
};
```

### CI/CD Logging

For CI/CD, collect and report summary metrics:

```typescript
const ciLogger = new MetricsLogger();

// After build
const metrics = ciLogger.getMetrics();

if (metrics.failedCompilations > 0) {
  console.error(`\n❌ ${metrics.failedCompilations} compilation failures`);
  process.exit(1);
} else {
  console.log(`\n✅ All compilations successful`);
  console.log(`   ${metrics.totalMemoBlocks} reactive scopes created`);
  console.log(`   ${metrics.totalMemoValues} values memoized`);
}
```
