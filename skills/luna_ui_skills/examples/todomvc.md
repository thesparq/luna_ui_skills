# TodoMVC Example

Source: `src/examples/todomvc/`

## File Structure

```
src/examples/todomvc/
├── moon.pkg
├── main.mbt
├── types.mbt
└── README.mbt.md
```

## Types (`types.mbt`)

```moonbit
///| Todo item
pub struct Todo {
  id : Int
  title : String
  completed : Bool
}

///| Filter mode
pub(all) enum Filter {
  All
  Active
  Completed
}
```

## Main Component (`main.mbt`)

```moonbit
///| TodoMVC application
fn todomvc() -> @luna.Node[E, A] {
  let todos = @luna.signal([
    { id: 1, title: "Learn Luna", completed: false },
    { id: 2, title: "Build an app", completed: false },
  ])
  let filter = @luna.signal(All)
  let input_value = @luna.signal("")
  let next_id = @luna.signal(3)

  // Derived: filtered todos
  let filtered = @luna.memo(fn() {
    let f = filter.get()
    todos.get().filter(fn(t) {
      match f {
        All => true
        Active => !t.completed
        Completed => t.completed
      }
    })
  })

  div(
    [("class", @luna.attr_static("todoapp"))],
    [
      // Header
      header(
        [("class", @luna.attr_static("header"))],
        [
          h1([text("todos")]),
          input(
            [
              ("class", @luna.attr_static("new-todo")),
              ("placeholder", @luna.attr_static("What needs to be done?")),
              ("value", @luna.attr_dynamic(fn() { input_value.get() })),
              ("autofocus", @luna.attr_static("")),
            ],
            on=events()
              .input(fn(e) {
                input_value.set(e.target().value())
              })
              .keydown(fn(e) {
                if e.key() == "Enter" && input_value.get().length() > 0 {
                  todos.update(fn(ts) {
                    ts.push({
                      id: next_id.get(),
                      title: input_value.get(),
                      completed: false,
                    })
                    ts
                  })
                  next_id.update(fn(id) { id + 1 })
                  input_value.set("")
                }
              }),
            children=[],
          ),
        ],
      ),

      // Todo list section
      section(
        [("class", @luna.attr_static("main"))],
        [
          ul(
            [("class", @luna.attr_static("todo-list"))],
            [
              @luna.for_each(
                fn() { filtered.get() },
                fn(todo, _index) {
                  li(
                    [("class", @luna.attr_dynamic(fn() {
                      if todo.completed { "completed" } else { "" }
                    }))],
                    [
                      div(
                        [("class", @luna.attr_static("view"))],
                        [
                          input(
                            [
                              ("class", @luna.attr_static("toggle")),
                              ("type", @luna.attr_static("checkbox")),
                              ("checked", @luna.attr_dynamic(fn() {
                                if todo.completed { "true" } else { "false" }
                              })),
                            ],
                            on=events().click(fn(_) {
                              let id = todo.id
                              todos.update(fn(ts) {
                                for i, t in ts {
                                  if t.id == id {
                                    ts[i] = { ...t, completed: !t.completed }
                                  }
                                }
                                ts
                              })
                            }),
                            children=[],
                          ),
                          label(
                            on=events().dblclick(fn(_) { /* edit */ }),
                            children=[text(todo.title)],
                          ),
                          button(
                            [("class", @luna.attr_static("destroy"))],
                            on=events().click(fn(_) {
                              todos.update(fn(ts) {
                                ts.filter(fn(t) { t.id != todo.id })
                              })
                            }),
                            children=[],
                          ),
                        ],
                      ),
                    ],
                  )
                },
              ),
            ],
          ),
        ],
      ),

      // Footer with filter
      footer(
        [("class", @luna.attr_static("footer"))],
        [
          span(
            [("class", @luna.attr_static("todo-count"))],
            [text_of(todos, fn(ts) {
              let active_count = ts.filter(fn(t) { !t.completed }).length()
              active_count.to_string() + " items left"
            })],
          ),
          ul(
            [("class", @luna.attr_static("filters"))],
            [
              li(on=events().click(fn(_) { filter.set(All) }),
                children=[text("All")]),
              li(on=events().click(fn(_) { filter.set(Active) }),
                children=[text("Active")]),
              li(on=events().click(fn(_) { filter.set(Completed) }),
                children=[text("Completed")]),
            ],
          ),
        ],
      ),
    ],
  )
}
```

## Key Points

- **`for_each`** with reference-based reconciliation for list rendering
- **`signal()`** for mutable state
- **`memo()`** for derived/computed values
- **Events via `events().click(...).keydown(...)`** DSL
- **Dynamic CSS classes** via `attr_dynamic()`
- **`text_of()`** shorthand for signal-to-text binding
