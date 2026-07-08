# Routing

## Route Definition (Core)

Routes are defined using URLPattern syntax:

```moonbit
let routes : Array[@routes.Routes] = [
  @routes.Page(
    path="/",
    component="home",
    title="Home",
    meta=[],
  ),
  @routes.Page(
    path="/user/:id",
    component="user_profile",
    title="User Profile",
    meta=[],
  ),
  @routes.Layout(
    segment="/dashboard",
    children=[
      @routes.Page(
        path="/",
        component="dashboard_home",
        title="Dashboard",
        meta=[],
      ),
      @routes.Page(
        path="/settings",
        component="dashboard_settings",
        title="Settings",
        meta=[],
      ),
    ],
    layout="dashboard_layout",
  ),
  @routes.Get(path="/api/users", handler="list_users"),
  @routes.Post(path="/api/users", handler="create_user"),
]
```

## URL Parameter Syntax

- `:param` — Named parameter (`/user/:id` matches `/user/42`)
- `[param]` — Optional segment (`/user/[id]` matches `/user/42` and `/user`)
- `[...catchAll]` — Catch-all (`/files/[...path]` matches `/files/a/b/c`)
- `[[...optionalCatchAll]]` — Optional catch-all

## Route Compilation

```moonbit
let compiled = @routes.compile(routes)
// Returns Array[CompiledRoutes]
```

## URL Matching

```moonbit
let match = @routes.match_url("/user/42?tab=profile", compiled)
// Returns RoutesMatch? with params, query, path

// Accessors
match.params()   // Array[(String, String)] — [("id", "42")]
match.query()    // Array[(String, String)] — [("tab", "profile")]
match.path()     // "/user/42"
match.route()    // CompiledRoutes reference
```

## Browser Router (Client-side SPA)

```moonbit
let router = @router.BrowserRouter::new(routes, base="/app")

// Navigation
router.navigate("/user/42")    // pushState
router.replace("/settings")    // replaceState

// Reactive state
let path_signal = router.path_signal()        // Signal[String]
let match_signal = router.match_signal()      // Signal[RoutesMatch?]

// Current values
let current_path = router.get_path()
let current_match = router.get_match()

// React to route changes
@luna.effect(fn() {
  match router.get_match() {
    Some(m) => render_component(m.component())
    None => render_not_found()
  }
})

// Cleanup
router.dispose()
```

## JS API (Router Lite)

```moonbit
// Minimal router for tree-shaking
@api_router_lite.create_router(routes)
@api_router_lite.navigate("/new-path")
@api_router_lite.current_path()
```

## SSR + CSR Integration

Routes defined in core can be used on both server and client:
1. **Server**: Resolve route to determine which component to SSR
2. **Client**: Same route definitions compiled and used by BrowserRouter

### With Sol Framework

```moonbit
// Server-side integration via sol_link
@sol_link.sol_link(
  href="/about",
  children=[text("About")],
)
```
