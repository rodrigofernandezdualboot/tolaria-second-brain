---
type: Note
status: Active
relationships:
  Type:
    - "[[note]]"
belongs_to: "[[software-architecture]]"
url: https://martinfowler.com/articles/feature-toggles.html
author: Pete Hodgson
_organized: true
---

# Feature Toggles

Reference summary of **Feature Toggles (aka Feature Flags)** by Pete Hodgson on
[martinfowler.com](https://martinfowler.com/articles/feature-toggles.html) (09 October 2017).

## What are Feature Toggles?

Feature Toggles (also called feature flags, feature bits, or feature flippers) are a set of
patterns that let teams **modify system behavior without changing code**. They allow new
functionality to be delivered rapidly but safely — code can ship to production while a feature
stays hidden, enabled dynamically only when ready.

A typical toggle separates a **Toggle Point** (where the decision is made in code) from a
**Toggle Router** (which resolves whether a feature is on or off), keeping conditional logic
out of the core code path.

## Categories of toggles

Toggles differ along two axes — **how long they live** and **how dynamic they are** — so it's
important to categorize them before choosing an implementation:

- **Release Toggles** — enable trunk-based development and continuous delivery by hiding
  in-progress or untested features. Transient and typically static.
- **Experiment Toggles** — perform A/B or multivariate testing; route users into cohorts to
  measure behavior. Highly dynamic, short-lived.
- **Ops Toggles** — control operational aspects of system behavior (e.g. kill-switches to
  disable a resource-intensive feature under load). Often long-lived, dynamic.
- **Permissioning Toggles** — change features or product experience for specific users
  (premium features, "champagne brunch" internal testing). Long-lived and very dynamic
  (per-request).

## Managing different categories

- **Static vs dynamic** — some toggles need only deploy-time configuration; others must flip
  at runtime or per request.
- **Long-lived vs transient** — a release toggle should be removed once a feature ships;
  a permissioning toggle may live for years. Match the management rigor to the lifespan.

## Implementation techniques

- **De-couple decision points from decision logic** — don't scatter toggle checks; route
  through a central toggle mechanism.
- **Inversion of Decision** — the feature-flagged component asks the router for a decision
  rather than the router reaching into it.
- **Avoid conditionals** — prefer strategy patterns/routing over sprinkling `if` statements.

## Toggle configuration

Approaches, roughly in order of increasing sophistication:

- Hardcoded configuration (commented lines / constants)
- Parameterized configuration (via request or env)
- Toggle configuration files
- Toggle configuration in an app database
- Distributed configuration systems (with UIs)

Guidance: **prefer static configuration** where possible; reserve dynamic routing for toggles
that genuinely need it. Support per-request overrides for testing and diagnostics.

## Working with feature-flagged systems

- **Expose the current toggle configuration** so it's observable.
- **Use structured configuration files** so toggles are discoverable and reviewable.
- **Manage different toggles differently** according to their category.
- Beware **validation complexity** — every toggle multiplies the combinations that must be
  tested; keep the count low.

## Where to place a toggle

- **At the edge** — good for permissioning and experiment toggles driven by request context.
- **In the core** — needed when the decision depends on internal state not available at the edge.

## Takeaway

Feature Toggles are powerful for decoupling deployment from release, but they introduce
complexity and carrying cost. Keep that in check with smart implementation practices,
appropriate configuration tooling, and — above all — by **constraining the number of toggles**
and removing transient ones as soon as they've served their purpose.
