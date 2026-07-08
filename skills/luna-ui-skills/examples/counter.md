# Counter Component Example

Source: `src/examples/hello_luna/`

## File Structure

```
src/examples/hello_luna/
├── moon.pkg
├── main.mbt
└── README.mbt.md
```

## `main.mbt` — Counter with Signals

```moonbit
///| Counter component with reactive signals
fn counter() -> @luna.Node[E, A] {
  let count = @luna.signal(0)

  div(
    [],
    [
      p(
        [("style", @luna.attr_static("font-size: 2em; font-weight: bold;"))],
        [text_of(count)],
      ),
      div(
        [],
        [
          button(
            [
              ("click", @luna.attr_handler(@luna.handler(fn(_) {
                count.update(fn(v) { v - 1 })
              }))),
            ],
            [@luna.text("-")],
          ),
          button(
            [
              ("click", @luna.attr_handler(@luna.handler(fn(_) {
                count.update(fn(v) { v + 1 })
              }))),
            ],
            [@luna.text("+")],
          ),
          button(
            [
              ("click", @luna.attr_handler(@luna.handler(fn(_) {
                count.set(0)
              }))),
            ],
            [@luna.text("Reset")],
          ),
        ],
      ),
    ],
  )
}

///| Main page component
fn page() -> @luna.Node[E, A] {
  div(
    [("class", @luna.attr_static("container"))],
    [
      h1([text("Luna Counter Demo")]),
      counter(),
      p([text("This counter uses reactive signals")]),
    ],
  )
}

///| Render entry
fn main {
  let result = @core.render_to_string(page())
  println(result.html)
}
```

## Island Version (for SSR + Client Hydration)

```moonbit
///| Counter as an island
fn counter_island() -> @luna.Node[E, A] {
  @luna.island({
    id: "counter",
    url: "/static/counter.js",
    state: """{"count":0}""",
    trigger: Load,
    children: [
      div(
        [("class", @luna.attr_static("counter"))],
        [
          span(
            [("class", @luna.attr_static("count-display"))],
            [text("0")],
          ),
          button(
            [
              ("click", @luna.attr_static("decrement")),
              ("class", @luna.attr_static("btn")),
            ],
            [text("-")],
          ),
          button(
            [
              ("click", @luna.attr_static("increment")),
              ("class", @luna.attr_static("btn")),
            ],
            [text("+")],
          ),
        ],
      ),
    ],
  })
}

///| Client-side hydration logic
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

## Key Points

- **Signals** drive reactivity: `signal()`, `get()`, `set()`, `update()`
- **DSL functions**: `div()`, `p()`, `button()`, `text()`, `text_of()`
- **Event handlers**: `events().click(fn(_) { ... })` or `@luna.attr_handler(@luna.handler(fn(_) { ... }))`
- **Islands**: Separate hydration boundary with own JS module
- **Actions**: `VAction("increment")` for declarative event handling
