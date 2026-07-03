---
type: Note
status: Active
relationships:
  Type:
    - "[[note]]"
belongs_to: "[[software-architecture]]"
url: https://martinfowler.com/microservices/
_organized: true
---

# Microservices Guide

Reference summary of the **Microservices Guide** on martinfowler.com, a hub for Martin Fowler and colleagues' material on the microservice architectural style.

> Note: captured from the [source](https://martinfowler.com/microservices/). Verify details against the article before relying on specifics.

## Definition

> In short, the microservice architectural style is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by fully automated deployment machinery. There is a bare minimum of centralized management of these services, which may be written in different programming languages and use different data storage technologies.
>
> — James Lewis and Martin Fowler (2014)

## Defining characteristics

Common characteristics of microservice architectures seen in the field:

- **Componentization via Services**
- **Organized around Business Capabilities**
- **Products not Projects**
- **Smart endpoints and dumb pipes**
- **Decentralized Governance**
- **Decentralized Data Management**
- **Infrastructure Automation**
- **Design for failure**
- **Evolutionary Design**

## When should we use microservices?

Any architectural style has trade-offs, and microservices are no exception. It is a useful architecture, but many — indeed most — situations would do better with a monolith.

### Benefits

- **Strong Module Boundaries** — Reinforce modular structure, particularly important for larger teams.
- **Independent Deployment** — Simple, autonomous services are easier to deploy and less likely to cause system-wide failures.
- **Technology Diversity** — Mix multiple languages, frameworks, and data-storage technologies.

### Costs

- **Distribution** — Distributed systems are harder to program; remote calls are slow and can fail.
- **Eventual Consistency** — Strong consistency is extremely hard in a distributed system, so everyone must manage eventual consistency.
- **Operational Complexity** — Requires a mature operations team to manage many services redeployed regularly.

## Key takeaways

- Microservices decompose an application into small, independently deployable services organized around business capabilities.
- Favor smart endpoints and dumb pipes, decentralized governance, and decentralized data management.
- Don't reach for microservices by default — the "microservice premium" means a monolith is often the better starting point.
- Weigh the trade-offs (module boundaries, independent deployment, tech diversity vs. distribution, eventual consistency, operational complexity) against your context.
