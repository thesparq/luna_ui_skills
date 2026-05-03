# Deployment

## Build for Production

```bash
# Full build
just build

# Or step by step:
moon build --target js --release src
moon build --target js --release src/js/api
moon build --target js --release src/js/api_signals
moon build --target js --release src/js/api_resource_lite
moon build --target js --release src/js/api_router_lite
pnpm turbo run @luna_ui/luna-loader#build
pnpm vite build
```

## Bundle Size Constraints

- **Loader** must be < 5KB (enforced by CI)
- Check sizes: `just size`
- Check against baseline: `just size-check`

## Integration with Sol (SSR/SSG Framework)

Luna is designed to work with the [Sol framework](https://github.com/mizchi/sol.mbt):

1. Use `sol.mbt` for SSR/SSG
2. Define Luna components as islands
3. Sol handles route rendering and island embedding

### Sol + Luna Link Component

```moonbit
// Server-side link
@sol_link.sol_link(
  href="/about",
  children=[text("About")],
)

// Client-side navigation handled by sol-nav.ts
```

## CSR Navigation (sol-nav)

The `sol-nav.ts` loader handles SPA navigation:
- Intercepts clicks on `[data-sol-link]` anchors
- Fetches HTML fragment from server
- Updates DOM with `setHTMLUnsafe` (preserves Declarative Shadow DOM)
- Implements stale-while-revalidate caching (5min TTL)
- Hover prefetch via `[data-sol-prefetch]`
- Handles browser back/forward via popstate

## Version Management

```bash
# Update version (patch/minor/major)
just vup patch
just vup minor
just vup major
just vup 0.5.0 --dry-run

# Generate changelog
just changelog v0.5.0
just changelog-preview   # Preview unreleased changes
```

## Output Structure

After build:
```
_build/js/release/build/mizchi/luna/
├── js/api/api.js           # Full MoonBit → JS API
├── js/api_signals/         # Signals-only
├── js/api_resource_lite/   # Resource-only
└── js/api_router_lite/     # Router-only

js/loader/dist/
├── loader.js               # Island loader (< 5KB)
├── wc-loader.js            # Web Components loader
└── sol-nav.js              # CSR navigation
```
