# architecture.md

**Goal:** Reader understands the system's shape — what the pieces are, how they
connect, and where data flows.

## Structure

1. **System Diagram** — Mermaid diagram showing major components and relationships.
   Include: entry points, core modules, data stores, external services.
   See `references/diagram-guide.md` for Mermaid patterns.
2. **Layer Map** — Identify the architectural layers (presentation, business logic,
   data access, infrastructure). Note which boundaries are clean vs. leaky.
3. **Data Flow** — Trace key operations from input to output. Scale with complexity:
   simple CRUD = 2 traces; systems with auth/payments/background jobs = 3-5 traces.
4. **Concurrency Model** — If applicable: threading, async, event loop, workers.
5. **Why This Matters** — General principles of structural architecture.

## Concepts to Link

`[[layered-architecture]]`, `[[hexagonal-architecture]]`, `[[monolith]]`,
`[[microservices]]`, `[[event-loop]]`, `[[MVC]]`, `[[CQRS]]`, etc.
