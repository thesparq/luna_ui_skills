# DOM API Reference

Module: `mizchi/luna/dom` (JS-only)

## Element DSL

```moonbit
pub fn div(id?, class?, style?, on?, ref?, attrs?, dyn_attrs?, children) -> DomNode
pub fn p(id?, class?, style?, on?, ref?, attrs?, dyn_attrs?, children) -> DomNode
pub fn button(id?, class?, style?, on?, ref?, attrs?, dyn_attrs?, children) -> DomNode
pub fn span(id?, class?, style?, on?, attrs?, dyn_attrs?, children) -> DomNode
// ... and more (generated in __generated.mbt)
```

## Events DSL

```moonbit
pub fn events() -> HandlerMap
// Chaining methods (each returns HandlerMap):
events()
  .click(MouseHandler)      .dblclick(MouseHandler)
  .input(InputHandler)      .change(ChangeHandler)
  .submit(FormHandler)      .keydown(KeyboardHandler)
  .keyup(KeyboardHandler)   .keypress(KeyboardHandler)
  .focus(FocusHandler)      .blur(FocusHandler)
  .mouseenter(MouseHandler) .mouseleave(MouseHandler)
  .mouseover(MouseHandler)  .mouseout(MouseHandler)
  .mousedown(MouseHandler)  .mouseup(MouseHandler)
  .mousemove(MouseHandler)  .wheel(WheelHandler)
  .contextmenu(MouseHandler)
  .dragstart(DragHandler)   .drag(DragHandler)
  .dragend(DragHandler)     .dragenter(DragHandler)
  .dragleave(DragHandler)   .dragover(DragHandler)
  .drop(DragHandler)
  .touchstart(TouchHandler) .touchmove(TouchHandler)
  .touchend(TouchHandler)   .touchcancel(TouchHandler)
  .pointerdown(PointerHandler) .pointermove(PointerHandler)
  .pointerup(PointerHandler)   .pointercancel(PointerHandler)
  .pointerenter(PointerHandler).pointerleave(PointerHandler)
  .scroll(Handler)          .resize(Handler)
  .compositionstart(Handler).compositionend(Handler)
```

## DOM Rendering

```moonbit
pub fn create_element(tag, attrs, children) -> DomNode
pub fn create_element_ns(ns, tag, attrs, children) -> DomNode
pub fn text_node(content: () -> String) -> DomNode
pub fn text_from_signal[T: Show](sig) -> DomNode
pub fn show(when, render_fn) -> DomNode
pub fn for_each[T](items, render_item) -> DomNode
pub fn index_each[T](items, render_item) -> DomNode
pub fn loading(when~, fallback~, render_fn) -> DomNode
pub fn portal(target~, children~) -> DomNode
pub fn portal_to_body(children) -> DomNode
pub fn portal_to_selector(selector, children) -> DomNode
pub fn portal_to_element_with_shadow(target, children) -> DomNode
pub fn portal_with_shadow(children) -> DomNode
pub fn mount(container, node) -> Unit
pub fn render(container, node) -> Unit
pub fn clear(container) -> Unit
pub use_style(css, root) -> Unit
pub fn events() -> HandlerMap
```

## Client Hydration

```moonbit
pub fn hydrate(container, node, recover_on_mismatch?, warn_on_mismatch?, logger?) -> HydrationResult
pub fn hydrate_with_dispose(container, node, ...) -> (HydrationResult, () -> Unit)
pub fn bind_actions_from_dom(container, dispatch) -> Unit
pub fn hydrate_with_actions[State](container, initial_state, render, update) -> Unit
pub enum HydrationResult { Success, Mismatch(String), Recovered(String) }
pub struct IslandContext { element, id, state_json }
pub fn hydrate_island(element, state, id, setup) -> Unit
pub fn IslandContext::signal_int(key, default) -> Signal[Int]
pub fn IslandContext::bind_actions(dispatch) -> Unit
pub fn IslandContext::on_update(selector, update_fn) -> Unit
```

## Browser Router

```moonbit
pub struct BrowserRouter { routes, base, current_path, current_match, dispose_popstate }
pub fn BrowserRouter::new(routes, base?) -> BrowserRouter
pub fn BrowserRouter::navigate(path) -> Unit
pub fn BrowserRouter::replace(path) -> Unit
pub fn BrowserRouter::dispose() -> Unit
pub fn BrowserRouter::get_path() -> String
pub fn BrowserRouter::get_match() -> RoutesMatch?
pub fn BrowserRouter::get_component() -> String?
pub fn BrowserRouter::get_navigate() -> (String) -> Unit
pub fn BrowserRouter::path_signal() -> Signal[String]
pub fn BrowserRouter::match_signal() -> Signal[RoutesMatch?]
```

## Portal

```moonbit
pub fn shadow_portal(children) -> DomNode
pub fn body_portal(children) -> DomNode
pub fn selector_portal(selector, children) -> DomNode
```

## SVG/MathML

```moonbit
pub let svg_ns: String = "http://www.w3.org/2000/svg"
pub let mathml_ns: String = "http://www.w3.org/1998/Math/MathML"
```

## AttrValue

```moonbit
pub(all) enum AttrValue {
  Static(String)
  Dynamic(() -> String)
  Handler((@js.Any) -> Unit)
}
```

## ElementRef

```moonbit
pub type ElementRef = (@js_dom.Element) -> Unit
```
