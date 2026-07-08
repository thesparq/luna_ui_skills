# Troubleshooting

## Common Issues

### Type Check Fails

```bash
# Check for JS target
moon check --target js

# Check all targets
moon check --target all

# Fix formatting
moon fmt
```

### Build Issues

```bash
# Clean and rebuild
moon clean
rm -rf _build target .turbo/cache
moon build --target js --release

# Check target-specific build:
moon build --target js --release src
```

### Hydration Mismatch

Luna's hydrator detects mismatches and can recover:
- `Sol:hk` attribute markers in HTML
- Mismatch → warning logged + automatic re-render (if `recover_on_mismatch=true`)
- Check for: extra whitespace, different attributes, nested structures

### Loader Not Firing

1. Verify `luna:url` attribute is present on island div
2. Check script module path resolves correctly
3. Console should show hydration log messages
4. `window.__LUNA_HYDRATE__` available for manual hydration
5. `window.__LUNA_SCAN__()` to force re-scan

### Bundle Size Too Large

```bash
# Check sizes
just size
just treeshake-size

# Compare with Preact
just preact-size
just preact-size-json

# Update baselines
just bundle-baseline
just treeshake-baseline
```

Common size culprits:
- MoonBit's `Double.to_string` (uses ryu algorithm, ~31KB) — use JS FFI `double_to_string()`
- Struct with many optional fields → use opaque JS object via FFI
- Dead code not eliminated → check imports

### Shadow DOM / WC Issues

- Verify Declarative Shadow DOM support in target browsers
- Use `setHTMLUnsafe` for SSR content (legacy: `innerHTML` fallback)
- `use_style()` uses `getRootNode()` to find the correct ShadowRoot
- CSS dedup by hash ID prevents duplicate injection

## Development Tools

```bash
# Watch mode for auto-rebuild
moon build --target js --watch

# Verbose test output
moon test -v

# Filter specific tests
moon test src/core/render --filter "Island*"

# Update snapshots
moon test --update

# Coverage
just coverage
```

## Debugging Tips

1. Enable hydration warnings: `warn_on_mismatch=true`
2. Use `console_log_island()` for hydration diagnostics
3. Check `sol:hk` attribute distribution in SSR output
4. Verify all event handler names are lowercase (DOM standard)
5. Check that `ComponentRef[T]` props implement `ToJson`

## Testing Strategy

| Layer | Tool | What to Test |
|-------|------|-------------|
| MoonBit Unit | `moon test --target js` | Pure logic, VNode construction, SSR output |
| Vitest | `pnpm vitest run` | DOM operations, hydration, browser APIs |
| E2E | Playwright (`just test-e2e`) | Full integration, real browser |
| Cross-platform | `just test-xplat` | Routes, render, serialize on all targets |

## CI Pipeline

```bash
just ci
```
Runs: `check → test-incremental → size-check → runtime-check → moonbench-check`
