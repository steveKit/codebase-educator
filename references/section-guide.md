# Section Writing Guide

Templates and guidance for each section of an educator brief.
Sections use Obsidian-compatible markdown with `[[wikilinks]]`.

---

## architecture.md

**Goal:** Reader understands the system's shape — what the pieces are, how they
connect, and where data flows.

### Structure

1. **System Diagram** — Mermaid diagram showing major components and their relationships.
   Include: entry points, core modules, data stores, external services.
2. **Layer Map** — Identify the architectural layers (presentation, business logic,
   data access, infrastructure). Note which boundaries are clean vs. leaky.
3. **Data Flow** — Trace 1-2 key operations from input to output. Show which
   modules touch the data and in what order.
4. **Concurrency Model** — If applicable: threading, async, event loop, workers.
5. **Why This Matters** — Explain the general principles of structural architecture.
   What makes a good layer boundary? Why does data flow direction matter?

### Concepts to Link

`[[layered-architecture]]`, `[[hexagonal-architecture]]`, `[[monolith]]`,
`[[microservices]]`, `[[event-loop]]`, `[[MVC]]`, `[[CQRS]]`, etc.

---

## technology-choices.md

**Goal:** Reader understands not just *what* was chosen but *why* — and what
the alternatives were.

### Structure

1. **Stack Summary** — Table of language, framework, database, tooling choices
2. **Decision Analysis** — For each major choice:
   - What was chosen
   - Likely rationale (infer from usage patterns, ecosystem, project needs)
   - Trade-offs accepted (what's gained, what's given up)
   - Alternatives that existed at the time
3. **Ecosystem Fit** — How well the choices work together. Are there friction
   points where two choices don't naturally integrate?
4. **Why This Matters** — Technology selection is an architectural decision.
   Explain evaluation criteria: maturity, community, performance characteristics,
   hiring pool, ecosystem compatibility.

### Concepts to Link

`[[build-vs-buy]]`, `[[boring-technology]]`, `[[polyglot-persistence]]`,
`[[framework-lock-in]]`, etc.

### External Links

Link to official docs/homepage for each technology in the stack summary table.
Include registry links (npm, PyPI, etc.) where applicable.

---

## design-patterns.md

**Goal:** Reader learns to recognize patterns in real code and evaluate whether
they're applied well.

### Structure

1. **Patterns Identified** — For each pattern found:
   - Name and category (creational, structural, behavioral, architectural)
   - Where it appears (specific files/modules)
   - How it's implemented — brief code walkthrough
   - Quality assessment — is it well-applied, forced, or incomplete?
2. **Pattern Interactions** — How patterns work together in this codebase
3. **Missing Patterns** — Situations where a known pattern would help but isn't used
4. **Anti-Patterns** — Patterns that are technically present but harmful in context
5. **Why This Matters** — Patterns are communication tools, not rules. When to
   apply them vs. when simplicity wins. The Gang of Four caveat: "prefer
   composition over inheritance" — but why?

### Concepts to Link

`[[factory-pattern]]`, `[[observer-pattern]]`, `[[strategy-pattern]]`,
`[[dependency-injection]]`, `[[middleware-pattern]]`, `[[anti-pattern]]`, etc.

---

## key-decisions.md

**Goal:** Reader learns to read between the lines — understanding decisions
that aren't documented but shaped everything.

### Structure

1. **Identified Decisions** — For each:
   - What the decision was
   - Evidence in the code (what reveals it)
   - Likely constraints that drove it (time, team size, scale requirements, etc.)
   - Consequences — how this decision rippled through the codebase
2. **Decision Dependencies** — Which decisions constrained later choices
3. **Reversibility Assessment** — Which decisions could still be changed vs.
   which are now load-bearing
4. **Why This Matters** — Software is shaped by decisions made under constraints.
   Learning to identify and evaluate these decisions — especially the invisible
   ones — is a core architectural skill.

### Concepts to Link

`[[accidental-complexity]]`, `[[essential-complexity]]`, `[[technical-debt]]`,
`[[architectural-decision-record]]`, `[[reversibility]]`, etc.

---

## gaps-vulnerabilities.md

**Goal:** Reader learns what "good enough" looks like and where the risks hide.

### Structure

1. **Architectural Gaps** — Missing components or patterns that a mature system
   would typically have (e.g., no caching layer, no circuit breaker, no rate limiting)
2. **Scaling Concerns** — What would break first under 10x load? 100x?
3. **Security Surface** — Observable security concerns (not a full audit —
   reference `[[security-audit]]` for that)
4. **Reliability Gaps** — Missing error handling, no retry logic, no graceful
   degradation, no health checks
5. **Operational Gaps** — Missing observability, no structured logging, no
   deployment strategy visible
6. **Specialist Audit Referrals** — Flag areas where a dedicated audit tool
   would provide deeper insight. Don't perform the audit — recognize the need
   and explain why it matters architecturally. Common referrals:

   | Observation | Referral | Architectural Lesson |
   |---|---|---|
   | User-facing HTML with no ARIA, skip-nav, or semantic structure | `/a11y-audit` — `[[accessibility]]` | Accessibility is a design constraint, not a retrofit |
   | Public website with no meta tags, no structured data, no semantic headings | `/seo-audit` — `[[seo-architecture]]` | SEO is an architectural concern when discoverability drives business value |
   | Third-party dependencies with unclear licenses | `/legal-audit` — `[[license-compliance]]` | Dependency choices carry legal obligations |
   | No input validation, secrets in code, no auth boundaries | `/security-audit` — `[[defense-in-depth]]` | Security is layered and must be designed in, not bolted on |

   Format each referral as:
   > "This project [observation]. This matters because [architectural principle].
   > For detailed analysis, see `/skill-name`.
   > See `[[concept]]` for the broader principle."

   Only include referrals where there's genuine evidence of a gap — don't
   list every audit tool for every project.
7. **Severity Assessment** — Which gaps matter now vs. which are future problems
8. **Why This Matters** — No system is complete. The skill is knowing which gaps
   to close now, which to accept, and which to monitor. Explain the concept of
   "appropriate engineering" — not every app needs five-nines reliability.
   Knowing *that* a specialist concern exists (accessibility, SEO, compliance)
   is an architectural skill even when the details belong to a specialist.

### Concepts to Link

`[[defense-in-depth]]`, `[[circuit-breaker]]`, `[[graceful-degradation]]`,
`[[observability]]`, `[[blast-radius]]`, `[[accessibility]]`,
`[[seo-architecture]]`, `[[license-compliance]]`, etc.

---

## dependencies.md

**Goal:** Reader learns to evaluate dependency decisions — when to rely on
third-party code and when to build.

### Structure

1. **Dependency Overview** — Count, categories (framework, utility, dev tooling)
2. **Philosophy Assessment** — Where does this project sit on the spectrum from
   "build everything" to "npm install everything"?
3. **Risk Analysis** — For notable dependencies:
   - Maintenance status (last publish date, open issues, bus factor)
   - How deeply coupled the project is to it (wrapper layer vs. deep integration)
   - What would replacing it cost
4. **Vendored / Forked Code** — Any signs of vendored dependencies or forks
5. **Dev Dependencies** — Build tooling, linting, testing framework choices
6. **Why This Matters** — Every dependency is a trust decision. Explain the
   trade-offs: speed vs. control, community vs. stability, breadth vs. depth.

### Concepts to Link

`[[dependency-inversion]]`, `[[vendor-lock-in]]`, `[[bus-factor]]`,
`[[supply-chain-security]]`, `[[build-vs-buy]]`, etc.

### External Links

Link to each notable dependency's official docs, GitHub repo, and registry page.
For the risk analysis entries, include the repo link so readers can check
maintenance status themselves.

---

## evolution.md

**Goal:** Reader learns to read a codebase's history and understand how
software systems evolve under real-world pressure.

### Structure

1. **Timeline Indicators** — Evidence of the project's age and evolution:
   - Git history patterns (if available)
   - Inconsistent code styles suggesting different eras
   - Deprecated APIs still in use alongside modern equivalents
   - Migration artifacts (old + new patterns coexisting)
2. **Growth Pattern** — How did the project grow? Organic sprawl vs. planned
   architecture? Big-bang rewrites vs. incremental migration?
3. **Sedimentary Layers** — Oldest code vs. newest code. What changed in approach?
4. **Migration Status** — Any in-progress migrations? How complete?
5. **Why This Matters** — All real codebases are palimpsests — old decisions
   visible under new ones. Learning to read these layers tells you what works
   long-term and what doesn't survive contact with reality.

### Concepts to Link

`[[strangler-fig-pattern]]`, `[[big-bang-rewrite]]`, `[[incremental-migration]]`,
`[[technical-debt]]`, `[[second-system-effect]]`, etc.

---

## testing-strategy.md

**Goal:** Reader learns what testing decisions reveal about a project's
priorities and engineering maturity.

### Structure

1. **Coverage Map** — What's tested and what isn't (by module/feature area)
2. **Testing Philosophy** — Unit-heavy? Integration-heavy? E2E? Property-based?
   What does the balance tell you?
3. **Test Quality** — Are tests testing behavior or implementation? Brittle or
   resilient to refactoring?
4. **Mocking Strategy** — How are dependencies handled in tests? Over-mocking?
   Real databases? Fakes?
5. **CI Integration** — Are tests automated? Fast enough for CI? Flaky test signs?
6. **Gaps** — Critical paths with no test coverage
7. **Why This Matters** — Testing strategy is an architectural decision that
   affects development speed, refactoring confidence, and bug detection.
   The testing pyramid (or trophy, or honeycomb) — what shape does this project
   have and what does it mean?

### Concepts to Link

`[[testing-pyramid]]`, `[[test-doubles]]`, `[[property-based-testing]]`,
`[[integration-testing]]`, `[[test-driven-development]]`, etc.

### External Links

Link to testing framework docs and any testing libraries used.

---

## if-starting-over.md

**Goal:** Turn analysis into actionable learning. What would you do differently
knowing what this codebase teaches?

### Structure

1. **Keep** — What this project got right that should be preserved
2. **Change** — What would be done differently and why
3. **Add** — What's missing that should be there from day one
4. **Drop** — What adds complexity without proportional value
5. **Architecture Sketch** — If you redesigned this system today, what would the
   high-level shape look like? (Mermaid diagram)
6. **Why This Matters** — Hindsight is a learning accelerator. The goal isn't to
   criticize but to internalize trade-off evaluation. Every "I'd do it differently"
   is a principle you can apply to your next project.

---

## learning-path.md

**Goal:** Give the reader an ordered reading list through the codebase.

### Structure

1. **Orientation Files** — Read these first (README, config, entry point)
2. **Core Domain** — The files that contain the actual business logic
3. **Infrastructure Layer** — How core domain connects to the outside world
4. **Extension Points** — Where the system is designed to be extended
5. **Deep Dives** — Complex areas worth studying carefully, with prereqs noted
6. **Estimated Time** — Rough time estimate for each phase of the reading list

Each entry: file path, what you'll learn from it, difficulty tag, prereqs.

---

## glossary.md

**Goal:** Remove vocabulary barriers to understanding the codebase.

### Structure

Alphabetical list. Each entry:
- **Term** — as used in the codebase
- **Meaning** — plain language definition in context
- **Where** — file(s) where it appears prominently
- **Concept link** — `[[concept-name]]` if it connects to a broader pattern

Include both domain terms (business language) and technical terms (framework-specific
jargon, architectural terminology) that a reader might not know.

---

## overview.md

**Goal:** Executive summary that serves as the entry point to the brief.

Written **last** because it summarizes all other sections.

### Structure

1. **One-Paragraph Summary** — What this is, what it does, why it's interesting
   to study
2. **Source Quality** — Prominent banner showing the overall quality rating
   (exemplary/solid/mixed/cautionary) with a one-sentence explanation. This
   tells the reader upfront how to interpret everything that follows.
   See `references/quality-assessment.md` for banner format.
3. **Key Takeaways** — 3-5 bullet points: the most valuable lessons from this
   codebase
4. **Architecture at a Glance** — Simplified Mermaid diagram (copied/condensed
   from architecture.md)
4. **Section Index** — Links to all sections with one-line descriptions and
   difficulty tags:
   - `[[<project>/architecture]]` — System structure and layers *(intermediate)*
   - `[[<project>/technology-choices]]` — Stack decisions *(beginner)*
   - ... etc.
5. **Concepts Introduced** — List of all `[[concept]]` pages referenced in
   this analysis, grouped by category
6. **Cross-Project Connections** — If other analyses exist in the vault,
   note which concepts this project shares with them

---

## resources.md

**Goal:** One-stop reference for all external links — official docs, repos,
registries, and supplementary guides.

Written **after all other sections** (except overview) because it consolidates
links embedded throughout.

### Structure

1. **Core Stack** — Table with Technology, Docs, Repository, Registry columns.
   One row per major technology (language, framework, database, runtime).
2. **Dependencies** — Table with Package, Purpose, Docs, Registry columns.
   Include all dependencies mentioned in `dependencies.md`. Group by category
   (runtime, dev tooling, testing) if the list is long (10+).
3. **Pattern References** — Table with Pattern, Reference columns. Link to
   authoritative explanations for each pattern discussed in `design-patterns.md`.
   Prefer language-specific guides over abstract theory.
4. **Further Reading** — Curated list of articles, talks, or guides that are
   genuinely relevant to understanding this codebase's approach. Not a generic
   reading list — every entry should connect back to something observed in the
   analysis. Format: `[Title](url) — why this is relevant`.

Omit any section or table column that would be empty.
