# Shared Section Writing Rules

These apply to every section file. Load this alongside the specific section
template — never load all 13 section templates at once.

## Wikilinking Section Headers

When a section header names a transferable concept — a design pattern, architectural
style, testing practice, or named principle — wikilink it in the body text near
the header. The header itself stays plain text (Obsidian doesn't render wikilinks
in headers well), but the first sentence under it should include the `[[concept-name]]`.

**Quick test:** If you could write a standalone concept page for it (with "How It
Works", "Trade-Offs", etc.) without referencing this specific project, it deserves
a wikilink.

This applies especially to:
- Pattern names in `design-patterns.md` headers
- Architectural styles in `architecture.md`
- Entries under "Missing Patterns" — the general concept still gets a page
- Anti-patterns — name them with a wikilink

The "Concepts to Link" lists in each section template are starting points,
not exhaustive. Any transferable idea that passes the test above should be linked.

## YAML Frontmatter

Every section starts with:

```yaml
---
source: <project-name>
section: <section-name>
difficulty: beginner | intermediate | advanced
quality-note: <how the source quality rating affects this section>
---
```

## Relevant Files Header

After YAML, list key source files (3-7) in a blockquote. Omit for overview,
if-starting-over, and resources sections.

With source URL: `` > **Relevant files:** [`path`](url) · [`path`](url) ``
Without: `` > **Relevant files:** `path` · `path` ``

## Code Examples

5-15 lines each. Always include file path + line numbers in a comment on
line 1 AND in a `<sub>` attribution below. Use `// ...` to elide.

With source URL:
````
```lang
// path#Lstart-Lend
<snippet>
```
<sub>[source](base-url/blob/branch/path#Lstart-Lend)</sub>
````

## Depth Minimums

| Category | Min words | Code examples |
|---|---|---|
| Implementation-heavy (architecture, design-patterns, key-decisions, testing-strategy, gaps-vulnerabilities) | 300 | 3-5 |
| Analytical (technology-choices, dependencies, evolution, if-starting-over) | 200 | 1-3 |
| Reference (learning-path, glossary, resources, overview) | Complete coverage | 0 |

## Quality Markers

For **mixed** and **cautionary** sources, tag every observation:
- **Worth emulating** / **Acceptable trade-off** / **Cautionary example** / **Anti-pattern**

## Inline Links

Every technology/framework/library is hyperlinked on first mention per section.
Pull URLs from the `url-index` register — never construct ad hoc.

## Closing

Every section (except resources) ends with a **"Why This Matters"** callout
explaining the transferable principle.
