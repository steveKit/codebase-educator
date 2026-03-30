# _<project-name>_overview.md

**Goal:** Executive summary — the entry point to the brief. Written **last**
because it summarizes everything.

Named with `_<project-name>_` prefix to sort first in Obsidian and
disambiguate across the vault.

## Frontmatter

```yaml
---
source: <project-name>
section: overview
difficulty: beginner
quality-note: <overall quality rating and one-line summary>
---
```

## Structure

1. **Source** — Prominent link at the top:
   - GitHub: `**Source:** [owner/repo](url)`
   - npm: `**Source:** [package](registry-url)`
   - Local: `**Source:** local`
2. **One-Paragraph Summary** — What, why, and why it's interesting to study
3. **Source Quality** — Banner with rating (exemplary/solid/mixed/cautionary)
   and one-sentence explanation. See `references/quality-assessment.md`.
4. **Key Takeaways** — 3-5 bullets: the most valuable lessons
5. **Architecture at a Glance** — Simplified Mermaid diagram (condensed
   from architecture.md)
6. **Section Index** — Links to all sections with one-line descriptions and
   difficulty tags. The full stack table lives in `[[project/technology-choices]]`
   — don't duplicate it here.
7. **Concepts Introduced** — All `[[concept]]` pages, grouped by category
8. **Cross-Project Connections** — Bidirectional. Phase 3 updates both this
   project and connected projects. Format per connection:
   concept name, how each project applies it, what the comparison teaches.
