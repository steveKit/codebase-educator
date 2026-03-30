# dependencies.md

**Goal:** Reader learns to evaluate dependency decisions — when to rely on
third-party code and when to build.

## Structure

1. **Dependency Overview** — Count, categories (framework, utility, dev tooling)
2. **Philosophy Assessment** — Where on the spectrum from "build everything"
   to "npm install everything"?
3. **Risk Analysis** — For notable dependencies:
   - Maintenance status (last publish date, open issues, bus factor)
   - How deeply coupled the project is (wrapper layer vs. deep integration)
   - Replacement cost
4. **Vendored / Forked Code** — Signs of vendored dependencies or forks
5. **Dev Dependencies** — Build tooling, linting, testing framework choices
6. **Why This Matters** — Every dependency is a trust decision. Trade-offs:
   speed vs. control, community vs. stability, breadth vs. depth.

## Concepts to Link

`[[dependency-inversion]]`, `[[vendor-lock-in]]`, `[[bus-factor]]`,
`[[supply-chain-security]]`, `[[build-vs-buy]]`, etc.

## External Links

Link each notable dependency's docs, GitHub repo, and registry page.
This section should be the most link-dense in the brief.
