# testing-strategy.md

**Goal:** Reader learns what testing decisions reveal about a project's
priorities and engineering maturity.

**Skip-eligible:** Yes — skip when no test files are found in the source.
The absence of tests is noted in `gaps-vulnerabilities.md` instead.

## Structure

1. **Coverage Map** — What's tested and what isn't (by module/feature area)
2. **Testing Philosophy** — Unit-heavy? Integration-heavy? E2E? Property-based?
   What does the balance tell you?
3. **Test Quality** — Testing behavior or implementation? Brittle or resilient?
4. **Mocking Strategy** — How are dependencies handled? Over-mocking? Real
   databases? Fakes?
5. **CI Integration** — Automated? Fast enough for CI? Flaky test signs?
6. **Gaps** — Critical paths with no test coverage
7. **Why This Matters** — Testing strategy is an architectural decision affecting
   development speed, refactoring confidence, and bug detection.

## Concepts to Link

`[[testing-pyramid]]`, `[[test-doubles]]`, `[[property-based-testing]]`,
`[[integration-testing]]`, `[[test-driven-development]]`, etc.

## External Links

Link testing framework docs and any testing libraries used.
