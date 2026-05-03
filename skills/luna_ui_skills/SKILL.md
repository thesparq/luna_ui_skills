---
name: luna_ui_skills
description: "Complete AI agent skill for the Luna UI library (mizchi/luna.mbt) - a fine-grained reactive UI library for MoonBit/JS with Island Architecture. Use when scaffolding, developing, refactoring, or debugging Luna UI projects. Covers VNode system, signals reactivity, SSR/hydration, DOM DSL, routing, CSS utilities, ARIA components, Web Components, and deployment."
metadata:
  author: thesparq
  version: "1.0.0"
---

# Skill: Luna UI Library

Setting up and managing any luna.mbt UI project using the MoonBit programming language.

Use when scaffolding or creating a new luna project, refactoring a luna project, generating code for a luna project, reviewing a luna project, or building and running a luna project.

## Reading Order

1. Getting started (quickest launch)
   - `docs/quickstart.md`
2. Architecture overview
   - `docs/architecture.md`
3. Core concepts
   - `docs/core-concepts.md`
4. Component development
   - `docs/components.md`
5. Routing
   - `docs/routing.md`
6. CSS utilities
   - `docs/css.md`
7. Deployment
   - `docs/deploy.md`
8. Troubleshooting
   - `docs/troubleshooting.md`

## Quick Links by Purpose

- Want to create a new component
  - `docs/components.md`
- Want to understand the Island architecture
  - `docs/architecture.md`
- Want to integrate with Sol SSR/SSG framework
  - `docs/deploy.md`
- Want to test your Luna app
  - `docs/troubleshooting.md`
- Want to check the public API reference
  - `api/core-api.md`
  - `api/dom-api.md`
  - `api/signal-api.md`

## Instructions

1. Understand the Island Architecture (partial hydration): SSR renders full HTML, client loader (~5KB) hydrates only interactive islands
2. Use VNode types `Node[E, A]` parameterized over event type and attribute type
3. Build reactive UIs with signals from `mizchi/signals` (SolidJS-compatible)
4. For SSR components, use `@luna` re-exports from `src/top.mbt`
5. For client components, use `@dom` DSL (`div()`, `events().click()`, etc.)
6. Create hydration boundaries via `island()`, `wc_island()`, or `client(ComponentRef, children)`
7. Define routes with `@routes.Routes` enum, compile with `@routes.compile()`
8. Use `BrowserRouter::new()` for client-side SPA navigation
9. Apply CSS via `use_style()`, CSS DSL in `@css`, or atomic CSS generation
10. Test at three levels: MoonBit unit (`moon test`), Vitest (DOM), Playwright (E2E)
