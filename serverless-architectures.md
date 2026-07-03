---
type: Note
status: Active
relationships:
  Type:
    - "[[note]]"
belongs_to: "[[software-architecture]]"
author: Mike Roberts
url: https://martinfowler.com/articles/serverless.html
_organized: true
---

# Serverless Architectures

Notes on Mike Roberts' in-depth article on serverless architecture, published on
[martinfowler.com](https://martinfowler.com/articles/serverless.html) (22 May 2018).

## What is Serverless?

Serverless architectures are application designs that remove much of the need for a
traditional always-on server component. The term covers two overlapping areas:

- **Backend as a Service (BaaS)** — incorporating third-party, cloud-hosted services to
  manage server-side logic and state (e.g. databases like Firebase/Parse, auth services like
  Auth0/AWS Cognito). Common in rich-client apps such as single-page and mobile apps.
- **Functions as a Service (FaaS)** — developer-written server-side logic run in stateless,
  event-triggered, ephemeral compute containers fully managed by a third party (e.g.
  AWS Lambda). This is the newer, hype-driving side and the article's main focus.

> "Serverless" is a misnomer: servers and processes still run somewhere — the org just
> outsources responsibility for that hardware and those processes to a vendor.

## Key characteristics of FaaS

- **Stateless** — no in-server state persists between invocations; state must live in an
  external store (e.g. a database or cache).
- **Ephemeral & event-triggered** — functions spin up in response to events (HTTP via an API
  gateway, queue messages, etc.) and may last only one invocation.
- **Cold starts** — startup latency when a function hasn't run recently.
- **Bounded execution duration** — providers cap how long a function may run.

## Benefits

- **Reduced operational cost** — pay only for actual usage; strong fit for occasional or
  inconsistent traffic.
- **Reduced development cost (BaaS)** — offload auth, databases, etc.
- **Automatic, fine-grained scaling** — scaling handled by the platform, not just cheaper but
  operationally simpler.
- **Simpler packaging/deployment** and faster time to market, enabling continuous
  experimentation.
- Potentially "greener" computing through better utilization.

## Drawbacks

**Inherent:**
- Vendor control, multitenancy issues, and vendor lock-in.
- Security concerns from a larger trust surface.
- Repetition of logic across client platforms.
- Loss of server-side optimizations and no in-server state.

**Implementation (may improve over time):**
- Configuration overhead, "DoS yourself" risks, execution-duration and startup-latency limits.
- Harder testing, debugging, deployment/versioning, discovery, and observability.
- Over-ambitious API gateway definitions.

## What Serverless is *not*

Distinguished from PaaS, containers, #NoOps, and "stored procedures as a service."

## The future

Roberts expects the drawbacks to be mitigated through better tooling, state-management
solutions, platform improvements, education, clearer vendor expectations, emerging patterns,
and abstractions/portable implementations over vendor-specific APIs.

## Takeaway

Serverless can significantly cut operational cost, complexity, and engineering lead time — at
the cost of greater reliance on vendor dependencies and (at time of writing) comparatively
immature supporting services.
