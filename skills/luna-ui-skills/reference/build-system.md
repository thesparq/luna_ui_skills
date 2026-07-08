# Build System

## MoonBit Build

### Module Configuration (`moon.mod.json`)

```json
{
  "name": "mizchi/luna",
  "version": "0.19.0",
  "source": "src",
  "preferred-target": "js",
  "deps": {
    "moonbitlang/async": "0.16.6",
    "moonbitlang/x": "0.4.40",
    "mizchi/js": "0.10.16",
    "mizchi/npm_typed": "0.1.13",
    "moonbitlang/parser": "0.2.5",
    "mizchi/signals": "0.6.4",
    "mizchi/x": "0.2.0"
  },
  "warn-list": "-6-unused_trait_bound"
}
```

### Package Configuration (`moon.pkg`)

Simple:
```
import {
  "mizchi/luna/core",
  "mizchi/luna/dom",
  "mizchi/signals",
}
```

Executable:
```
import {
  "mizchi/luna",
  "mizchi/luna/dom/dsl",
}
options("is-main": true)
```

Test imports:
```
import {
  "mizchi/luna" @luna,
  "mizchi/luna/dom" @dom,
} for "test"
```

### Build Targets

| Target | Backend | Use Case |
|--------|---------|----------|
| `js` | JavaScript | Browser, Node.js |
| `wasm-gc` | WebAssembly GC | Cross-platform core |
| `native` | Native | CLI tools, testing |

The `core/` package targets `"all"`; `dom/` and `js/` target `"js"` only.

### Build Commands

```bash
# Type check
moon check --target js

# Build release
moon build --target js --release src

# Build debug (with sourcemaps)
moon build --target js -g src

# Watch mode
moon build --target js --watch

# Clean
moon clean

# Test
moon test --target js
moon test -v                             # Verbose
moon test dir --filter "pattern"         # Filter
moon test --update                       # Update snapshots
moon test --target all src/core/routes   # Cross-platform
```

## JavaScript/TypeScript Build

### TurboRepo Pipeline (`turbo.json`)

```json
{
  "pipeline": {
    "build": { "dependsOn": ["^build"] },
    "test:moonbit": {},
    "test:vitest": {}
  }
}
```

### Vite (`vite.config.ts`)

Used for bundling the demo app and Vitest browser tests.

### Package Workspaces

- `js/luna/` — `@luna_ui/luna` (npm package)
- `js/loader/` — `@luna_ui/luna-loader` (npm package)

## Just Task Runner

```bash
# Development
just check            # moon check --target js
just fmt              # moon fmt
just build-moon       # Build all MoonBit targets
just build-loader     # Build loader via turbo
just build            # Full build

# Testing
just test-moonbit     # MoonBit unit tests
just test-vitest      # Vitest (node + browser)
just test-e2e         # Playwright E2E
just test-xplat       # Cross-platform tests
just retest           # Force (no cache)

# CI
just ci               # Full CI pipeline

# Bundle analysis
just size             # Bundle sizes
just size-check       # Loader < 5KB check
just treeshake-size   # Tree-shaken sizes
just preact-size      # Preact comparison

# Coverage
just coverage         # All coverage
just coverage-moonbit # MoonBit coverage only
```
