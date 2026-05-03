# Router Example

Source: `src/examples/browser_router/`

## File Structure

```
src/examples/browser_router/
├── moon.pkg
├── main.mbt
└── README.mbt.md
```

## Route Definitions

```moonbit
///| Application routes
let routes : Array[@routes.Routes] = [
  @routes.Page(
    path="/",
    component="home",
    title="Home",
    meta=[("description", "Home page")],
  ),
  @routes.Page(
    path="/about",
    component="about",
    title="About",
    meta=[("description", "About page")],
  ),
  @routes.Page(
    path="/users/:id",
    component="user_profile",
    title="User Profile",
    meta=[],
  ),
  @routes.Layout(
    segment="/settings",
    children=[
      @routes.Page(
        path="/profile",
        component="settings_profile",
        title="Profile Settings",
        meta=[],
      ),
      @routes.Page(
        path="/account",
        component="settings_account",
        title="Account Settings",
        meta=[],
      ),
    ],
    layout="settings_layout",
  ),
]
```

## Browser Router Setup

```moonbit
///| Main application
fn app() -> @luna.Node[E, A] {
  let router = @router.BrowserRouter::new(routes, base="/app")

  div(
    [],
    [
      // Navigation
      nav(
        [],
        [
          a(
            on=events().click(fn(_) { router.navigate("/") }),
            children=[text("Home")],
          ),
          a(
            on=events().click(fn(_) { router.navigate("/about") }),
            children=[text("About")],
          ),
          a(
            on=events().click(fn(_) { router.navigate("/users/42") }),
            children=[text("User 42")],
          ),
        ],
      ),

      // Route content
      div(
        [("id", @luna.attr_static("content"))],
        [
          @luna.show(
            condition=fn() { router.get_match() is Some(_) },
            child=fn() {
              match router.get_component() {
                Some("home") => home_page()
                Some("about") => about_page()
                Some("user_profile") => {
                  let match = try! router.get_match()
                  let user_id = match.param("id").unwrap_or("unknown")
                  user_profile(user_id)
                }
                Some("settings_profile") => settings_profile()
                Some("settings_account") => settings_account()
                _ => not_found()
              }
            },
          ),
        ],
      ),
    ],
  )
}

///| Component renderer with reactive routing
fn render_current_route(router : @router.BrowserRouter) -> @luna.Node[E, A] {
  // Reactively re-renders when route changes
  let _current_path = router.get_path() // Track dependency

  match router.get_component() {
    Some("home") => home_page()
    Some("about") => about_page()
    Some("user_profile") => {
      let match = try! router.get_match()
      let user_id = match.param("id").unwrap_or("unknown")
      user_profile(user_id)
    }
    Some("settings_profile") => settings_profile()
    Some("settings_account") => settings_account()
    _ => not_found()
  }
}
```

## Page Components

```moonbit
///| Home page
fn home_page() -> @luna.Node[E, A] {
  div([("class", @luna.attr_static("page"))], [
    h1([text("Home")]),
    p([text("Welcome to the Luna SPA example")]),
  ])
}

///| About page
fn about_page() -> @luna.Node[E, A] {
  div([("class", @luna.attr_static("page"))], [
    h1([text("About")]),
    p([text("This is a single-page application built with Luna")]),
  ])
}

///| User profile with route param
fn user_profile(user_id : String) -> @luna.Node[E, A] {
  div([("class", @luna.attr_static("page"))], [
    h1([text("User Profile")]),
    p([text("User ID: " + user_id)]),
  ])
}
```

## Using Nav Signal (Reactive)

```moonbit
///| Using match_signal for reactive rendering
fn reactive_content(router : @router.BrowserRouter) -> @luna.Node[E, A] {
  let match_sig = router.match_signal()

  @luna.show(
    condition=fn() { match_sig.get() is Some(_) },
    child=fn() {
      let m = try! match_sig.get()
      match m.route().component {
        "home" => home_page()
        "about" => about_page()
        _ => not_found()
      }
    },
  )
}
```

## URL Path Segments

```moonbit
///| Extract a route parameter
fn RoutesMatch::param(self : @routes.RoutesMatch, name : String) -> String? {
  for p in self.params {
    if p.0 == name {
      return Some(p.1)
    }
  }
  None
}

///| Get query parameter
fn RoutesMatch::query_param(self : @routes.RoutesMatch, name : String) -> String? {
  for q in self.query {
    if q.0 == name {
      return Some(q.1)
    }
  }
  None
}
```

## Key Points

- **Declarative routes**: `@routes.Page`, `@routes.Layout`
- **BrowserRouter**: Wraps History API + popstate
- **Reactive routing**: Access path/match via signals or getters
- **Type-safe params**: `RoutesMatch` with `params` and `query` arrays
- **URLPattern syntax**: `:param`, `[...catchAll]`, `[[...optional]]`
