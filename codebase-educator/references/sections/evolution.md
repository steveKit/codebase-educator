# evolution.md

**Goal:** Reader learns to read a codebase's history and understand how
software systems evolve under real-world pressure.

**Skip-eligible:** Yes — skip when <50 commits AND <6 months history AND
no migration artifacts. Young projects with no evolution story shouldn't
have this section padded.

## Structure

1. **Timeline Indicators** — Evidence of age and evolution:
   - Git history patterns (if available)
   - Inconsistent code styles suggesting different eras
   - Deprecated APIs alongside modern equivalents
   - Migration artifacts (old + new patterns coexisting)
2. **Growth Pattern** — Organic sprawl vs. planned architecture?
   Big-bang rewrites vs. incremental migration?
3. **Sedimentary Layers** — Oldest code vs. newest. What changed in approach?
4. **Migration Status** — Any in-progress migrations? How complete?
5. **Why This Matters** — All real codebases are palimpsests — old decisions
   visible under new ones. Reading these layers teaches what works long-term.

## Concepts to Link

`[[strangler-fig-pattern]]`, `[[big-bang-rewrite]]`, `[[incremental-migration]]`,
`[[technical-debt]]`, `[[second-system-effect]]`, etc.
