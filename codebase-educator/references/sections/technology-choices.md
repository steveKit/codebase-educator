# technology-choices.md

**Goal:** Reader understands not just *what* was chosen but *why* — and what
the alternatives were.

## Structure

1. **Stack Summary** — Table with linked technologies. Every name is a hyperlink.
   Pull from the `url-index` register.

   ```markdown
   | Category | Choice | Purpose | Links |
   |---|---|---|---|
   | Language | [Lang](homepage) | purpose | [Docs](url) |
   | Framework | [Framework](homepage) | purpose | [Docs](url) . [registry](url) . [GitHub](url) |
   ```

   Links column uses `.` separators for multiple links in one cell.

2. **Decision Analysis** — For each major choice: what was chosen, likely
   rationale, trade-offs accepted, alternatives that existed.
3. **Ecosystem Fit** — How well choices work together. Friction points?
4. **Why This Matters** — Technology selection is an architectural decision.
   Evaluation criteria: maturity, community, performance, hiring pool, ecosystem.

## Concepts to Link

`[[build-vs-buy]]`, `[[boring-technology]]`, `[[polyglot-persistence]]`,
`[[framework-lock-in]]`, etc.
