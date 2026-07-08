# Core API Reference

Module: `mizchi/luna/core` (imported as `@core`)

## Types

```moonbit
pub(all) enum Node[E, A]           // Virtual DOM node
pub(all) enum Attr[E, A]           // Attribute value
pub struct VElement[E, A]          // Element node
pub struct VIsland[E, A]           // Hydration island
pub struct VWcIsland[E, A]         // Web Components island
pub struct VInternalRef[E, A]      // Type-safe internal reference
pub struct VAsync[E, A]            // Async node
pub struct VErrorBoundary[E, A]    // Error boundary
pub struct VSwitch[E, A]           // Switch/Match
pub struct MatchCase[E, A]         // Match case
pub struct EventHandler[E]         // Event handler wrapper
pub(all) enum TriggerType          // Hydration trigger
pub(all) struct ComponentRef[T]    // Component reference
pub type StaticDomNode             // = Node[Unit, String]
pub type StaticDomAttr             // = Attr[Unit, String]
```

## Node Constructors

| Function | Signature | Description |
|----------|-----------|-------------|
| `h` | `(String, Array[(String, Attr[E, A])], Array[Node[E, A]]) -> Node[E, A]` | Create element |
| `text` | `(String) -> Node[E, A]` | Static text |
| `text_dyn` | `(() -> String) -> Node[E, A]` | Dynamic text |
| `fragment` | `(Array[Node[E, A]]) -> Node[E, A]` | Fragment |
| `raw_html` | `(String) -> Node[E, A]` | Raw HTML |
| `show` | `(condition~, child~) -> Node[E, A]` | Conditional |
| `for_each` | `(render~ : () -> Array[...]) -> Node[E, A]` | List |
| `component` | `(render~) -> Node[E, A]` | Component wrapper |
| `island` | `(VIsland[E, A]) -> Node[E, A]` | Island node |
| `wc_island` | `(VWcIsland[E, A]) -> Node[E, A]` | WC island |
| `async_` | `(VAsync[E, A]) -> Node[E, A]` | Async node |
| `error_boundary` | `(VErrorBoundary[E, A]) -> Node[E, A]` | Error boundary |
| `switch_` | `(VSwitch[E, A]) -> Node[E, A]` | Switch |
| `match_case` | `(when~, render~) -> MatchCase[E, A]` | Match case |
| `internal_ref` | `(String, String, trigger~, wc~, styles~, children~) -> Node[E, A]` | Internal ref |

## Attr Constructors

| Function | Signature | Description |
|----------|-----------|-------------|
| `attr_static` | `(A) -> Attr[E, A]` | Static attribute |
| `attr_dynamic` | `(() -> A) -> Attr[E, A]` | Dynamic attribute |
| `attr_handler` | `(EventHandler[E]) -> Attr[E, A]` | Event handler |
| `action` | `(String) -> Attr[E, A]` | Declarative action |
| `handler` | `((E) -> Unit) -> EventHandler[E]` | Create handler |
| `event_handler` | `((E) -> Unit) -> EventHandler[E]` | SSR handler |

## SSR Functions (core/render)

```moonbit
pub fn render_to_string(node, preload~) -> SSRResult
pub fn render_to_string_with_hydration(node) -> String
pub fn generate_preload_tags(urls) -> String
pub fn to_abstract(node) -> AbstractNode
pub fn render_attrs_to(sb, attrs) -> Unit
```

## Routes (core/routes)

```moonbit
pub(all) enum Routes { Page, Layout, Get, Post }
pub struct CompiledRoutes { pattern, param_names, component, layouts, kind, ... }
pub struct RoutesMatch { route, params, query, path }
pub fn compile(routes, base~) -> Array[CompiledRoutes]
pub fn match_url(url, routes) -> RoutesMatch?
```

## Stream Render (core/stream_render)

```moonbit
pub fn render_to_stream(node, chunks~) -> Unit
```

## Serialize (core/serialize)

```moonbit
pub(open) trait Serializable { to_state_value, from_state_value }
pub struct ResumableState { values, next_id }
pub fn register_signal(state, signal) -> Int
pub fn restore_signal(state, signal, id) -> Bool
pub fn create_resumable_signal(state, initial) -> (Signal[T], Int)
pub fn resume_signal(state, id, initial) -> Signal[T]
```

## AbstractNode (core/render)

```moonbit
pub(all) enum AbstractNode { Text, Element, Component, Fragment, RawHtml }
pub(all) enum RenderMode { SSROnly, Hydration(HydrationTrigger), ClientOnly, ServerDefer }
pub(all) enum HydrationTrigger { Load, Idle, Visible, Media(String) }
```
