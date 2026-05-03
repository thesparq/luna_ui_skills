# Island Architecture Deep Dive

## What is Island Architecture?

Island Architecture (also called "partial hydration" or "islands") renders static HTML on the server and selectively hydrates only interactive components on the client. This is the approach used by Qwik, Astro, and Luna.

### Key Benefits

1. **Minimal JavaScript**: Only interactive components send JS to the client
2. **Progressive Enhancement**: Static content works without JS
3. **Performance**: Tiny loader (~5KB), no framework runtime needed for static content
4. **SEO**: Full HTML at initial page load

## Luna's Implementation

### SSR (Server)

The server produces complete HTML with hydration boundaries:

```html
<!--luna:island:counter url=/static/counter.js trigger=load-->
<div luna:id="counter" luna:url="/static/counter.js" luna:state="{&quot;count&quot;:0}">
  <button data-action-click="decrement">-</button>
  <span class="count-display">0</span>
  <button data-action-click="increment">+</button>
</div>
<!--/luna:island:counter-->
```

### Loader (Client, < 5KB)

The loader is a framework-agnostic dispatcher:

```typescript
// js/loader/src/loader.ts (61 lines)
// - Scans for [luna:url] elements
// - Parses luna:state from attribute or <script> reference
// - Dynamically imports island modules
// - Calls hydrate(el, state, id) on each island
// - Supports triggers: load, idle, visible, media
// - MutationObserver for dynamic islands
```

### Island Module (Client)

Each island is a separate JS module with a `hydrate` export:

```moonbit
///| Counter island hydration
pub fn hydrate(element : @js.Any, state : @js.Any, id : String) -> Unit {
  @dom.hydrate_island(element, state, id, fn(ctx) {
    let count = ctx.signal_int("count", 0)

    ctx.bind_actions(fn(action) {
      match action {
        "increment" => count.set(count.get() + 1)
        "decrement" => count.set(count.get() - 1)
        _ => ()
      }
    })

    ctx.on_update(".count-display", fn() {
      count.get().to_string()
    })
  })
}
```

## Three Island Types

### 1. Traditional Island (`VIsland`)
```moonbit
@luna.island({
  id: "my-island",
  url: "/static/comp.js",
  state: json_string,
  trigger: Load,
  children: [...],
})
```

### 2. Web Components Island (`VWcIsland`)
```moonbit
@luna.wc_island({
  name: "my-counter",
  url: "/static/wc-counter.js",
  styles: ":host { display: block }",
  state: json_string,
  trigger: Load,
  children: [...],
})
```

### 3. Type-safe Internal Ref (`VInternalRef`)
```moonbit
let cref = @luna.component_ref("/static/counter.js", { count: 0 })
// Or for WC:
let wc_cref = @luna.wc_component_ref("/static/wc-counter.js", { count: 0 })

// In server component:
@static.client(cref, [text("Loading...")])
// Dispatch: Island or WcIsland based on cref.wc
```

## Hydration Triggers

| Trigger | Mechanism | Use Case |
|---------|-----------|----------|
| `Load` | `DOMContentLoaded` | Standard interactive components |
| `Idle` | `requestIdleCallback` | Below-the-fold, analytics |
| `Visible` | `IntersectionObserver` | Lazy loading, infinite scroll |
| `Media(query)` | `matchMedia` | Responsive components |
| `None` | Manual via `__LUNA_HYDRATE__` | User-initiated hydration |

## CSR Navigation (sol-nav)

For SPA-like navigation, `sol-nav.ts` handles:
1. Intercepts `[data-sol-link]` clicks
2. Fetches HTML fragment from server (`X-Sol-Fragment: true`)
3. Updates DOM via `setHTMLUnsafe()` (preserves Declarative Shadow DOM)
4. Disposes old islands, scans for new ones
5. Implements stale-while-revalidate cache strategy

## MoonBit API Integration

```moonbit
// Re-exported at src/top.mbt for easy access:
@luna.island(...)
@luna.wc_island(...)
@luna.internal_ref(...)
@luna.component_ref(...)
@luna.wc_component_ref(...)
@luna.client(cref, children)

// SSR entry points:
@core.render_to_string(vnode, preload=true)
@core.render_to_string_with_hydration(vnode)
```

## Web Components Island Flow

1. **SSR**: Element with Declarative Shadow DOM:
   ```html
   <my-counter luna:wc-url="/static/counter.js">
     <template shadowrootmode="open">
       <style>/* component styles */</style>
       <button>0</button>
     </template>
   </my-counter>
   ```
2. **Client**: wc-loader imports the module, which defines the custom element (`customElements.define`)
3. **Hydration**: Browser automatically upgrades Declarative Shadow DOM, WC constructor adds event listeners
