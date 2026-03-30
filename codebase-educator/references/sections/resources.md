# resources.md

**Goal:** One-stop reference for all external links. Written **second-to-last**
(before overview) because it consolidates links from all other sections.

## Writing Method

This is a **collection task, not a reconstruction task**:
1. Grep the written section files for markdown links (`[text](url)`)
2. Deduplicate and categorize into the structure below
3. Fill gaps (dependencies mentioned in prose but never linked inline)
4. Pull from the `url-index` register — do not re-research URLs

## Structure

1. **Core Stack** — Table: Technology, Docs, Repository, Registry.
   One row per major technology.
2. **Dependencies** — Table: Package, Purpose, Docs, Registry.
   Group by category (runtime, dev, testing) if 10+.
3. **Pattern References** — Table: Pattern, Reference.
   Prefer language-specific guides over abstract theory.
4. **Further Reading** — Curated list. Every entry connects back to an
   observation. Format: `[Title](url) -- why this is relevant`.

Omit any section or column that would be empty.
