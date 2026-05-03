# Components

## Element DSL (DOM)

Luna provides a SolidJS-inspired DSL for creating reactive DOM elements:

```moonbit
///| Basic element creation
div(
  class="my-class",
  style="color: red;",
  on=events().click(fn(e) { /* handler */ }).input(fn(e) { /* handler */ }),
  ref=fn(el) { /* element ref callback */ },
  attrs=[("data-id", Attr::AttrString("42"))],
  dyn_attrs=[("aria-expanded", Dynamic(fn() { if open.get() { "true" } else { "false" } }))],
  children=[text("Hello"), p([text("world")])],
)
```

### Event Handlers via `events()` DSL

```moonbit
// Method chaining for event handlers
events()
  .click(fn(e) { /* MouseHandler */ })
  .input(fn(e) { /* InputHandler */ })
  .keydown(fn(e) { /* KeyboardHandler */ })
  .submit(fn(e) { /* FormHandler */ })
  .focus(fn(e) { /* FocusHandler */ })
  .scroll(fn(e) { /* Handler */ })
  .drag(fn(e) { /* DragHandler */ })
  .touchstart(fn(e) { /* TouchHandler */ })
  .pointerdown(fn(e) { /* PointerHandler */ })
```

Available event types: click, dblclick, input, change, submit, keydown, keyup, keypress, focus, blur, mouseenter, mouseleave, mouseover, mouseout, mousedown, mouseup, mousemove, wheel, contextmenu, compositionstart, compositionend, drag, dragstart, dragend, dragenter, dragleave, dragover, drop, touchstart, touchmove, touchend, touchcancel, pointerdown, pointermove, pointerup, pointercancel, pointerenter, pointerleave, scroll, resize.

## Control Flow Components

### Show (Conditional Rendering)

```moonbit
let show_count = @luna.signal(true)

@luna.show(
  condition=fn() { show_count.get() },
  child=fn() { text("Content is visible") },
)
```

### For (List Rendering)

```moonbit
let items = @luna.signal(["a", "b", "c"])

@luna.for_each(
  fn() { items.get() },
  fn(item, index) {
    p([text(item)])
  },
)
```

Reference-based reconciliation (SolidJS-style). Uses `moveBefore` API for state-preserving node moves.

### Index (Index-based List)

```moonbit
// Optimized for primitive lists where identity = position
@luna.index_each(
  fn() { items.get() },
  fn(item_getter, index) {
    text_node(fn() { item_getter().to_string() })
  },
)
```

### Loading (Stale-while-revalidate)

```moonbit
@luna.loading(
  when=fn() { resource_is_pending },
  fallback=fn() { text("Loading...") },
  fn() { render_content() },
)
```

### Portal

```moonbit
// Render to document.body
@luna.portal_to_body([modal_content()])

// Render to CSS selector
@luna.portal_to_selector("#modal-root", [modal_content()])

// Render with Shadow DOM
@luna.portal_with_shadow([isolated_component()])
```

### Switch/Match

```moonbit
@luna.switch_(
  cases=[
    @luna.match_case(fn() { tab.get() == 1 }, fn() { tab1_content() }),
    @luna.match_case(fn() { tab.get() == 2 }, fn() { tab2_content() }),
  ],
  fallback=fn() { text("Select a tab") },
)
```

### ErrorBoundary

```moonbit
@luna.error_boundary(
  children=fn() { might_throw() },
  fallback=fn(err, reset) {
    div([text("Error: " + err.to_string())])
  },
)
```

## CSS Injection

```moonbit
let css = ".my-button { color: blue; }"

// Inside Shadow DOM's root, or document.head
use_style(css, element_ref)
```

## Web Components Integration

### Define Custom Element

```moonbit
// From js/api/define.mbt
@api.define(
  "my-counter",
  fn() { /* element setup */ },
)
```

### WC Island in SSR

```moonbit
let cref = @luna.wc_component_ref(
  "/static/wc-counter.js",
  { count: 0 },
)
client(cref, [text("Loading WC...")])
```

## ARIA/APG Component Library

Luna provides 20+ WAI-ARIA Authoring Practices Guide components in `src/x/components/`:

### Button
```moonbit
@cmp.button([text("Save")])
@cmp.toggle_button(ToggleState::Pressed, [text("Mute")])
@cmp.menu_button(Menu, false, controls="dropdown", [text("Options")])
```

### Other components: accordion, alert, breadcrumb, checkbox, combobox, dialog, disclosure, landmarks, link, listbox, meter, radio, slider, spinbutton, switch, table, tabs, toolbar, tooltip, treeview.

Three layers:
- **headless/** — Logic-only (state management, keyboard nav)
- **styled/** — Styled variants with CSS
- **APG docs** — WAI-ARIA pattern documentation

### Headless example (Accordion)

```moonbit
let state = @headless.AccordionState::new()
@headless.accordion(
  state,
  [
    @headless.accordion_item("panel-1", "Header 1", [text("Content 1")]),
    @headless.accordion_item("panel-2", "Header 2", [text("Content 2")]),
  ],
)
```
