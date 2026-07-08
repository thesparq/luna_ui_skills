# Architecture

## Island Architecture

Luna implements **Island Architecture** (also known as partial hydration):
- **Server** renders full HTML with hydration boundaries using `luna:*` attributes and HTML comments
- **Client** loads a tiny loader (~5KB) that scans for islands and hydrates them independently
- Components are either static (SSR-only) or interactive (hydration-required)

## Three Island Types

### 1. VIsland (Traditional)
Attribute-based islands using `luna:*` markers:
- `luna:id` — unique identifier
- `luna:url` — script URL for island implementation
- `luna:state` — serialized JSON state
- `luna:client-trigger` — hydration trigger (load/idle/visible/media)
- Wrapped in HTML comments: `<!--luna:island:id-->...<!--/luna:island:id-->`

### 2. VWcIsland (Web Components)
Uses Declarative Shadow DOM for SSR:
- Custom element with `luna:wc-url`, `luna:wc-state`, `luna:wc-trigger`
- `<template shadowrootmode="open">` for Shadow DOM content
- CSS isolation via Shadow DOM

### 3. VInternalRef (Type-safe)
Generated from `ComponentRef[T]`:
- Type-safe props via `ToJson` trait
- Dispatches to Island or WcIsland based on `wc` flag
- Recommended API: use `server_dom.client(cref, children)` function

## Rendering Pipeline

```
VNode (src/core/vnode.mbt)
  ├── render_to_string()     → HTML string (SSR)
  ├── render_to_stream()     → Streaming chunks (SSR)
  ├── to_abstract()          → AbstractNode (framework-agnostic)
  └── render_vnode_to_dom()  → DOM nodes (CSR)
```

## Directory Structure

```
src/
├── core/             # Environment-independent core (all targets)
│   ├── vnode.mbt     # VNode types: Node[E, A], VElement, VIsland, etc.
│   ├── render/       # SSR rendering, AbstractNode conversion
│   ├── routes/       # Route definitions & URL matching
│   ├── serialize/    # Qwik-like state serialization
│   └── stream_render/ # Streaming SSR
├── dom/              # DOM operations (JS-only)
│   ├── dsl.mbt       # Element DSL: div, p, button, events(), use_style
│   ├── render.mbt    # DOM rendering: create_element, show, for_each, portal
│   ├── reconcile.mbt # Reference-based reconciliation
│   ├── client/       # Client hydration
│   ├── router/       # Browser History API router
│   └── static/       # Server DOM helpers (island, sol_link)
├── js/               # JS-specific implementations
│   ├── resource/     # Signal/Resource (re-exports from mizchi/signals)
│   └── api/          # MoonBit → JS FFI API exports
└── x/                # Experimental modules
    ├── components/   # ARIA/APG-compliant components
    ├── css/          # Atomic CSS utilities
    └── stella/       # Island shard generation
js/
├── luna/             # @luna_ui/luna (npm package, SolidJS-compatible API)
└── loader/           # @luna_ui/luna-loader (island dispatch, >5KB)
```

## Cross-Platform Design

- **`core/`** targets `"all"` (js, wasm-gc, native) — no browser APIs
- **`dom/`** targets `"js"` only — browser DOM operations
- **`js/resource`** targets `"js"` — signal integration
- Parameterized over event type `E` and attribute type `A`

## Bundle Size Optimization

- Loader < 5KB (enforced by CI)
- JS FFI used extensively to avoid MoonBit's Double.to_string (ryu algorithm ~31KB)
- HandlerMap as opaque JS object (avoids MoonBit struct bloat)
- Multiple npm entrypoints for optimal tree-shaking

## State Serialization (Qwik-like)

- `ResumableState` container with sequential IDs
- `Serializable` trait for type-safe signal serialization
- JSON embedding in `<script>` tags with `data-resumable-state`
- XSS prevention via `escape_json_for_script()` and `escape_json_for_attr()`
