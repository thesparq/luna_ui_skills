# Core Concepts

## VNode System

Luna uses a virtual node (VNode) system parameterized over two type variables:

```moonbit
pub(all) enum Node[E, A] {
  Element(VElement[E, A])
  Text(String)
  DynamicText(() -> String)
  Fragment(Array[Node[E, A]])
  Show(condition~ : () -> Bool, child~ : () -> Node[E, A])
  For(render~ : () -> Array[Node[E, A]])
  Component(render~ : () -> Node[E, A])
  Island(VIsland[E, A])
  WcIsland(VWcIsland[E, A])
  Async(VAsync[E, A])
  ErrorBoundary(VErrorBoundary[E, A])
  Switch(VSwitch[E, A])
  InternalRef(VInternalRef[E, A])
  RawHtml(String)
}
```

- `E`: Event type (browser: `DomEvent`, SSR: `Unit`)
- `A`: Attribute value type (web: `String`, TUI: custom)

## Type Aliases

```moonbit
pub type StaticDomNode = Node[Unit, String]
pub type StaticDomAttr = Attr[Unit, String]
```

## Attribute Types

```moonbit
pub(all) enum Attr[E, A] {
  VStatic(A)                    // Static value
  VDynamic(() -> A)             // Dynamic (signal-backed)
  VHandler(EventHandler[E])     // Event handler
  VAction(String)               // Declarative action name
}
```

## Hydration Triggers

```moonbit
pub(all) enum TriggerType {
  Load      // Hydrate on DOMContentLoaded
  Idle      // Hydrate via requestIdleCallback
  Visible   // Hydrate via IntersectionObserver
  Media(String) // Hydrate when media query matches
  None      // Manual only via __LUNA_HYDRATE__
}
```

## Render Mode

```moonbit
pub(all) enum RenderMode {
  SSROnly                          // No client JS
  Hydration(HydrationTrigger)      // SSR + client hydration
  ClientOnly                       // Client-only (placeholder on server)
  ServerDefer                      // Server Islands
}
```

## Component Reference (Type-safe Islands)

```moonbit
pub(all) struct ComponentRef[T] {
  url : String       // Path to client JS
  props : T          // Props (must be ToJson)
  wc : Bool          // Use Web Components?
  trigger : TriggerType
}
```

Usage in server component:
```moonbit
let cref = @luna.component_ref("/static/counter.js", { initial_count: 42 })
client(cref, [text("Loading...")])
```

## Signals Reactivity

Signals are re-exported from `mizchi/signals`:

```moonbit
let count = @luna.signal(0)            // Create signal
count.get()                              // Read (tracks dependency)
count.set(5)                             // Write
count.update(fn(v) { v + 1 })            // Update with function

let doubled = @luna.memo(fn() { count.get() * 2 })  // Derived signal

let dispose = @luna.effect(fn() {        // Side effect
  println(count.get())
})

@luna.on_cleanup(fn() { /* cleanup */ }) // Cleanup registration
@luna.on_mount(fn() { /* on mount */ })  // Mount hook
```

## Resource API

```moonbit
let resource = @luna.resource(fn(resolve, reject) {
  // Async fetch
})

match resource {
  @luna.Resource::Pending => render_loading()
  @luna.Resource::Success(value) => render_content(value)
  @luna.Resource::Failure(error) => render_error(error)
}
```

## Context API (SolidJS-style)

```moonbit
let ThemeContext = @luna.create_context(fn() { "light" })

// Provide
@luna.provide(ThemeContext, "dark", children_fn)

// Consume
let theme = @luna.use_context(ThemeContext)
```
