# CSS Utilities

Luna provides atomic CSS generation utilities in `src/x/css/`.

## CSS DSL

```moonbit
let my_css = css(
  styles(
    ".my-button",
    [("color", "blue"), ("font-size", "16px")],
  ),
  styles(
    ".my-button:hover",
    [("color", "darkblue")],
  ),
)
```

## Pseudo-classes

```moonbit
let hover_styles = css(
  styles(
    ".btn",
    [("background", "blue")],
  ),
  hover(
    styles(
      ".btn",
      [("background", "darkblue")],
    ),
  ),
  focus(
    styles(
      ".btn",
      [("outline", "2px solid gold")],
    ),
  ),
  active(
    styles(
      ".btn",
      [("background", "navy")],
    ),
  ),
)
```

Available pseudo-class helpers: `hover`, `focus`, `active`, `visited`, `disabled`, `checked`, `focus_visible`, `focus_within`, `first_child`, `last_child`, `nth_child`, `not`.

## Media Queries

```moonbit
let responsive = css(
  dark(
    styles("body", [("background", "#111"), ("color", "#eee")]),
  ),
  at_md(
    styles(".container", [("flex-direction", "row")]),
  ),
  at_lg(
    styles(".container", [("max-width", "960px")]),
  ),
)
```

Breakpoints: `at_sm` (640px), `at_md` (768px), `at_lg` (1024px), `at_xl` (1280px), `at_2xl` (1536px).

## Style Registry & Hashing

Uses DJB2 hash algorithm to generate short class names (e.g., `_a1b2c3d`):

```moonbit
let registry = @css.Registry::new()
let class_name = registry.register(".my-class { color: red }")
// class_name => "_a1b2c3d"
```

## CSS Generation

```moonbit
// Minified (default)
let minified_css = @css.generate(css_rules)

// Pretty-printed
let pretty_css = @css.generate_pretty(css_rules)

// Full (no dedup)
let full_css = @css.generate_full(css_rules)
```

## CLI Tool (via `just`)

```bash
# Extract CSS from source files
just extract-css src

# Minify CSS file
just minify-css input.css

# Inject CSS into HTML
just inject-css index.html --src styles.css

# All CSS commands
just luna css --help
```

## CSS Optimizer

The CSS optimizer (in `src/x/css/optimizer/`) performs:
- Property merging
- Shorthand conversion
- Declaration reordering
- Co-occurrence analysis

## Inline CSS (use_style)

For Shadow DOM or document.head injection:

```moonbit
use_style(
  ".my-comp { color: red; }",
  element_ref,
)
```

Deduplicates by CSS hash (DJB2: `5381` base). Injects into nearest ShadowRoot or `document.head`.

## Benchmark

```bash
# CSS generation benchmark with configurable scale
just bench-css all
```
