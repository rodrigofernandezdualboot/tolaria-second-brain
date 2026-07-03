---
type: Note
status: Active
relationships:
  Type:
    - "[[note]]"
belongs_to: "[[software-architecture]]"
url: https://martinfowler.com/articles/micro-frontends.html
_organized: true
---

# Micro Frontends

Reference summary of **Micro Frontends** by Cam Jackson (Thoughtworks) on martinfowler.com, describing how to break up frontend monoliths into smaller, independently deliverable pieces.

> Note: captured from the [source](https://martinfowler.com/articles/micro-frontends.html) (19 June 2019). Verify details against the article before relying on specifics.

## Definition

> An architectural style where independently deliverable frontend applications are composed into a greater whole.

The technique applies the ideas that made microservices popular on the backend to frontend development, letting many teams work simultaneously on a large, complex product while it still appears to customers as a single cohesive experience.

## Benefits

- **Incremental upgrades** — Strangle the old monolith piece by piece, upgrading dependencies, stacks, and features case-by-case instead of via a risky full rewrite.
- **Simple, decoupled codebases** — Smaller, more cohesive and maintainable codebases with clearer boundaries.
- **Independent deployment** — Each micro frontend has its own pipeline and can be built, tested, and deployed independently.
- **Autonomous teams** — Decoupled ownership lets teams work end-to-end on a slice of the product without stepping on each other.

## Integration approaches

Micro frontends can be composed in several ways:

- **Server-side template composition**
- **Build-time integration** (e.g. publishing each as a package)
- **Run-time integration via iframes**
- **Run-time integration via JavaScript** (most flexible; each micro frontend exposes an entry point)
- **Run-time integration via Web Components**

## Cross-cutting concerns

- **Styling** — Avoid clashes via conventions (BEM), CSS Modules, CSS-in-JS, or shadow DOM.
- **Shared component libraries** — Promote visual consistency and reduce duplicated effort.
- **Cross-application communication** — Prefer loose coupling: custom events, callbacks passed by the container, or shared state via the address/route.
- **Backend communication** — Often paired with a Backend-for-Frontend (BFF) per micro frontend.
- **Testing** — Same strategies as any web app, plus contract tests at integration boundaries.

## Downsides

- **Payload size** — Independent builds can duplicate common dependencies, increasing bytes users download.
- **Environment differences** — Developing in isolation risks behavior that differs from the integrated production container.
- **Operational and governance complexity** — More repos, pipelines, and moving parts; increased autonomy can fragment how teams work.

## Key takeaways

- Micro frontends extend microservice thinking to the browser: independently deliverable frontends composed into one product.
- Emphasize the emergent attributes (incremental upgrades, decoupled codebases, independent deployment, autonomous teams) over any specific implementation.
- Run-time integration via JavaScript is the most flexible composition approach.
- Costs are real (payload duplication, environment drift, operational/governance overhead) but generally manageable when the technique fits the context.
