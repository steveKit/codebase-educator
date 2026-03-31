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
depth: deep | standard | light | skip
quality-note: <how the source quality rating affects this section>
skip-reason: <only present when depth is skip>
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

## Depth Tiers

Each section's depth is set in `quality.yaml` during Phase 1.5, based on
how much genuine material the source has for that section. Read
`section-notes.<section>.depth` before writing.

| Tier | Words | Code samples | Diagrams | Approach |
|---|---|---|---|---|
| **deep** | 500+ | 4-6 | 2-4 | Full treatment — thorough prose, comparison tables, multiple traces |
| **standard** | 300-500 | 2-4 | 1-2 | Solid coverage, enough to teach the key points |
| **light** | 150-300 | 0-2 | 0-1 | Essentials only — don't pad thin material |
| **skip** | stub | 0 | 0 | One-line explanation in frontmatter, no real content |

Word counts are guidance, not targets. A concise section that covers
everything is always better than a padded one. The tier tells you
"how much is here to say," not "how much you must write."

If the depth tier feels wrong as you write (e.g., you're assigned
`standard` but keep finding rich material), adjust upward. The
assessment is a starting point, not a constraint.

## Quality Markers

For **mixed** and **cautionary** sources, tag every observation:
- **Worth emulating** / **Acceptable trade-off** / **Cautionary example** / **Anti-pattern**

## Inline Links

Every technology/framework/library is hyperlinked on first mention per section.
Pull URLs from the `url-index` register — never construct ad hoc.

## Closing

Every section (except resources) ends with a **"Why This Matters"** callout
explaining the transferable principle.
