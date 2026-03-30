# Sections Manifest Register

**File:** `/tmp/educator-<name>/sections.yaml`
**Written by:** Phase 2 section writers (each appends their entry)
**Read by:** Phase 2.5 (concept sweep), Phase 3 (concept collection),
resources.md writer

Tracks what each section writer produced — concepts wikilinked, external
URLs used, key topics covered. Enables Phase 3 to collect concepts without
re-reading all 13 section files.

## Schema

```yaml
# /tmp/educator-<name>/sections.yaml
project-name: expressjs--express

sections:
  architecture:
    status: done
    word-count: 450
    code-examples: 4
    concepts-linked:
      - layered-architecture
      - middleware-pattern
      - event-loop
      - chain-of-responsibility
    urls-used:
      - https://expressjs.com/en/guide/using-middleware.html
      - https://nodejs.org/docs/latest/api/http.html
    mermaid-diagrams: 2

  technology-choices:
    status: done
    word-count: 320
    concepts-linked:
      - boring-technology
      - build-vs-buy
    urls-used:
      - https://expressjs.com
      - https://www.npmjs.com/package/express
      # ...
    mermaid-diagrams: 0

  # ... one entry per section
```

## How It's Built

Each section writer, after finishing its file:
1. Counts words (excluding YAML frontmatter and code blocks)
2. Counts code fence blocks
3. Extracts all `[[concept-name]]` wikilinks (deduplicated)
4. Extracts all `[text](url)` links (deduplicated)
5. Counts mermaid diagrams
6. Appends its entry to this file

## How It's Consumed

- **Phase 2.5 (Concept Sweep):** Reads all `concepts-linked` lists to know
  what's already linked, then scans section headers for gaps
- **Phase 3:** Collects the union of all `concepts-linked` across sections —
  this is the complete concept list without re-grepping 13 files
- **resources.md writer:** Collects the union of all `urls-used` to build
  the consolidated resources page
- **Phase 5 (Report):** Reads word counts and code example counts for summary
