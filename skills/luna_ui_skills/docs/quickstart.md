# Quickstart

## Prerequisites

- MoonBit toolchain (`moon` CLI)
- Node.js >= 18
- pnpm (for JS packages)
- `just` (task runner)

## Create a new project

```bash
# Using the existing Luna repo as template (or start from scratch)
moon new my-luna-app
cd my-luna-app
moon add mizchi/luna
moon add mizchi/signals
moon add mizchi/js
```

## Project structure for a Luna app

```
my-luna-app/
├── moon.mod.json
├── moon.pkg          # Root package (main entry)
├── main.mbt          # Main application file
├── justfile          # Task definitions
├── js/               # JS/TS packages
│   ├── package.json
│   └── src/
│       └── client.ts # Client entry (island hydration)
└── index.html        # Entry HTML
```

## Minimal example: Counter

### MoonBit server component (`main.mbt`)

```moonbit
fn main {
  // Build VNode tree
  let vnode = div(
    [h1([text("Hello Luna")]),
     button([text("Click me")])],
  )

  // SSR: render to HTML string
  let result = render_to_string(vnode)
  println(result.html)
}
```

### Client component with signals

```moonbit
///| Counter component with reactive signals
fn counter() -> @luna.Node[E, A] {
  let count = @luna.signal(0)

  div(
    [],
    [
      p([text_of(count)]),
      button(
        [
          ("click", @luna.attr_handler(@luna.handler(fn(_) {
            count.set(count.get() + 1)
          }))),
        ],
        [@luna.text("+1")],
      ),
    ],
  )
}
```

## Build & Run

```bash
# Type check
moon check --target js

# Build for JS target
moon build --target js --release

# Run tests
moon test --target js

# Format code
moon fmt
```

## Common tasks (via `just`)

```bash
just check         # Type check
just build-moon    # Build MoonBit
just test-moonbit  # Run MoonBit tests
just fmt           # Format
```
