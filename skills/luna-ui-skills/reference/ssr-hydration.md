# SSR & Hydration

## SSR Pipeline

### Basic SSR

```moonbit
let vnode = div([h1([text("Hello")])])
let result = @core.render_to_string(vnode)
// result.html => "<div><h1>Hello</h1></div>"
// result.preload_urls => [] (unless preload=true)
```

### SSR with Hydration Markers

```moonbit
let html = @core.render_to_string_with_hydration(vnode)
// Adds sol:hk="0", sol:hk="1", etc. for DOM matching
```

### SSR with Islands

```moonbit
let island = @luna.island({
  id: "counter-1",
  url: "/static/counter.js",
  state: """{"count": 0}""",
  trigger: Load,
  children: [text("0")],
})

let result = @core.render_to_string(island)
// <!--luna:island:counter-1 url=/static/counter.js trigger=Load-->
// <div luna:id="counter-1" luna:url="/static/counter.js" luna:state="...">
//   0
// </div>
// <!--/luna:island:counter-1-->
```

### SSR with Preload URLs

```moonbit
let result = @core.render_to_string(page, preload=true)
let preload_tags = @core.generate_preload_tags(result.preload_urls)
// <link rel="modulepreload" href="/static/counter.js">
```

## Streaming SSR

```moonbit
// Async streaming renderer (js/stream_renderer)
@stream.render_to_stream(vnode, fn(chunk) {
  // Send chunk to client
})
```

## HTML Comment Markers

| Marker | Node Type | Example |
|--------|-----------|---------|
| `t:N` | DynamicText | `<!--t:0-->text<!--/t-->` |
| `s:N` | Show | `<!--s:0-->content<!--/s-->` |
| `f:N` | For | `<!--f:0--><!--fi:0-->item<!--/fi:0--><!--/f-->` |
| `i:N` | Island | `<!--i:0--><!--/i-->` |
| `wc:N` | WcIsland | `<!--wc:0--><!--/wc-->` |
| `a:N` | Async | `<!--a:0-->fallback<!--/a-->` |
| `eb:N` | ErrorBoundary | `<!--eb:0-->content<!--/eb-->` |
| `sw:N` | Switch | `<!--sw:0-->matched<!--/sw-->` |

## Hydration Algorithm

1. SSR produces HTML with markers (`sol:hk` attributes, comment markers)
2. Client loads `@luna_ui/luna-loader` (~5KB)
3. Loader scans for `[luna:url]` elements
4. For each island:
   a. Parse `luna:state` attribute
   b. Dynamically import module from `luna:url`
   c. Call module's `hydrate(el, state, id)` function
5. Within hydrated islands, `hydrate_node()` traverses VNode tree:
   a. Finds matching DOM nodes via `sol:hk` cache (O(1) lookup)
   b. Attaches event handlers, dynamic attribute effects
   c. Sets up Show/For/component reactive effects

## Loader Architecture

```typescript
// Simplified loader flow:
1. Parse <script type="luna/json"> state preloads
2. DOMContentReady → scan for [luna:url] elements
3. MutationObserver → scan dynamically added elements
4. For each island:
   - Check trigger type (load/idle/visible/media)
   - Import island module
   - Call hydrate function
```

Globals available:
- `window.__LUNA_STATE__` — State map
- `window.__LUNA_HYDRATE__` — Manual hydration
- `window.__LUNA_SCAN__` — Force re-scan
- `window.__LUNA_UNLOAD__` — Unload all
- `window.__LUNA_CLEAR_LOADED__` — Clear loaded set

## Hydration Mismatch Recovery

```moonbit
let result = @dom.hydrate(
  container,
  vnode,
  recover_on_mismatch=true,
  warn_on_mismatch=true,
)
// On mismatch: clears container and re-renders (recovery)
```

## Web Components Hydration

```typescript
// wc-loader.ts handles WC islands
// Requirements:
// 1. SSR: <my-counter luna:wc-url="/static/counter.js">
//          <template shadowrootmode="open">...</template>
//        </my-counter>
// 2. Client: wc-loader imports and upgrades custom elements
// 3. WC defines hydrate function called by loader
```

## State Serialization (Qwik-style)

```moonbit
let state = @serialize.ResumableState::new()
let (count_sig, count_id) = @serialize.create_resumable_signal(state, 0)

// Serialize
let json = state.to_json()
// [0]

// Embed in HTML
<script type="luna/json" id="state-1">[0]</script>

// Restore on client
let loaded = @serialize.ResumableState::from_json(json)
let count = @serialize.resume_signal(loaded, count_id, 0)
```

## Sol SSR Integration

Sol framework handles:
1. Route resolution → component selection
2. SSR rendering with Luna's render_to_string
3. Island embedding via `client(ComponentRef, children)`
4. sol-link component generation for CSR navigation
5. sol-nav.ts handles client-side page transitions
