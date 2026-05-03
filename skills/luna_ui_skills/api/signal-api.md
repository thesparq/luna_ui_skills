# Signals & Resource API Reference

Module: `mizchi/luna/js/resource` (re-exports from `mizchi/signals`)

## Signal API

```moonbit
pub fn signal[T](value: T) -> Signal[T]
// Create a reactive signal

pub fn Signal::get(self) -> T
// Read current value (tracks as dependency in active effect)

pub fn Signal::peek(self) -> T
// Read without tracking dependency

pub fn Signal::set(self, value: T) -> Unit
// Set new value (notifies subscribers)

pub fn Signal::update(self, f: (T) -> T) -> Unit
// Update with function
```

## Computed / Memo

```moonbit
pub fn memo[T](f: () -> T) -> Signal[T]
// Create derived signal (lazy, caches, notifies on change)

pub fn computed[T](f: () -> T) -> Signal[T]
// Alias for memo
```

## Effects

```moonbit
pub fn effect(f: () -> Unit) -> () -> Unit
// Create effect (runs on next microtask, returns dispose)

pub fn render_effect(f: () -> Unit) -> () -> Unit
// Immediate/synchronous effect (runs during rendering)

pub fn effect_when(condition: () -> Bool, f: () -> Unit) -> Unit
// Effect that runs when condition becomes true

pub fn effect_once(f: () -> Unit) -> Unit
// Effect that runs exactly once and disposes itself
```

## Batching

```moonbit
pub fn batch_start() -> Unit
pub fn batch_end() -> Unit
pub fn batch(f: () -> Unit) -> Unit
// Batch multiple signal updates into one notification

pub fn untracked[T](f: () -> T) -> T
// Run function without tracking dependencies
```

## Lifecycle

```moonbit
pub fn on_cleanup(f: () -> Unit) -> Unit
// Register cleanup (runs when owner is disposed)

pub fn on_mount(f: () -> Unit) -> Unit
// Register mount callback (runs once when owner is mounted)

pub fn create_root[T](f: () -> T) -> T
// Create reactive root (for manual disposal)

pub fn create_root_with_dispose[T](f: () -> T) -> (T, () -> Unit)
// Create reactive root with dispose function

pub fn get_owner() -> Owner?
pub fn run_with_owner[T](owner: Owner, f: () -> T) -> T
pub fn has_owner() -> Bool
pub fn register_disposer(f: () -> Unit) -> Unit
pub fn register_owner_cleanup(f: () -> Unit) -> Unit
```

## Utility Signals

```moonbit
pub fn combine2[A, B, C](a: Signal[A], b: Signal[B], f: (A, B) -> C) -> Signal[C]
pub fn combine3[A, B, C, D](...) -> Signal[D]
pub fn combine4[A, B, C, D, E](...) -> Signal[E]

pub fn all[T](signals: Array[Signal[T]]) -> Signal[Array[T]]
pub fn any[T](signals: Array[Signal[Bool]]) -> Signal[Bool]

pub fn select[T](signals: Array[Signal[T]], index: Signal[Int]) -> Signal[T]
pub fn flatten[T](signal: Signal[Signal[T]]) -> Signal[T]

pub fn watch[T](signal: Signal[T], f: (T) -> Unit) -> () -> Unit
pub fn watch_immediate[T](signal: Signal[T], f: (T) -> Unit) -> () -> Unit

pub fn previous[T](signal: Signal[T]) -> Signal[T?]
pub fn previous_with_initial[T](signal: Signal[T], initial: T) -> Signal[T]
```

## Resource API

```moonbit
pub enum AsyncState[T] {
  Pending
  Success(T)
  Failure(String)
}

pub struct Resource[T] { ... }

pub fn resource[T](fetcher: ((T) -> Unit, (String) -> Unit) -> Unit) -> Resource[T]
// Create async resource

pub fn deferred[T]() -> (Resource[T], (T) -> Unit, (String) -> Unit)
// Create deferred resource (manually resolved)

pub fn resource_resolved[T](value: T) -> Resource[T]
// Create pre-resolved resource

pub fn resource_rejected[T](error: String) -> Resource[T]
// Create pre-rejected resource

pub fn Resource::get(self) -> AsyncState[T]
// Get current state (tracks dependency)

pub fn Resource::peek(self) -> T?
// Get value without tracking

pub fn Resource::refetch(self) -> Unit
// Trigger refetch

pub fn Resource::is_pending(self) -> Bool
pub fn Resource::is_success(self) -> Bool
pub fn Resource::is_failure(self) -> Bool
pub fn Resource::value(self) -> T?
pub fn Resource::error(self) -> String?
```

## Context API

```moonbit
pub struct Context[T] { ... }

pub fn create_context[T](default: () -> T) -> Context[T]
pub fn provide[T](context: Context[T], value: T, children: () -> Node) -> Node
pub fn use_context[T](context: Context[T]) -> T
```

## Debounced

```moonbit
pub fn debounced[T](signal: Signal[T], delay_ms: Int) -> Signal[T]
// Creates a debounced signal
```

## JS API (SolidJS-compatible)

Available via `@luna_ui/luna` npm package:

```typescript
// API entry: index.ts
createSignal<T>(value): Signal<T>
createEffect(fn): () => void
createRenderEffect(fn): () => void
createMemo<T>(fn): Accessor<T>
createResource<T>(fetcher): ResourceAccessor<T>
createDeferred<T>(): [ResourceAccessor<T>, resolve, reject]
createStore<T>(initial): [T, SetStoreFunction<T>]
createContext<T>(default): Context<T>
useContext<T>(context): T

// Components
Show, For, Index, Switch, Match, Loading, Portal, Provider, Fragment

// DOM
text, render, mount, show, jsx, jsxs, events, forEach

// Router
createRouter, routerNavigate, routerReplace

// Utilities
batch, onCleanup, onMount, createRoot, getOwner, untrack, on, mergeProps, splitProps, debounced
```
