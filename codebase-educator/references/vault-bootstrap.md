# Vault Bootstrap

Templates for the vault-level files created on first run.

---

## _concepts/_registry.yaml (Concept Index)

```yaml
# _concepts/_registry.yaml
# Auto-maintained by codebase-educator. Do not edit manually.
# Maps concept names to the projects that reference them.
# Used for cross-project connection lookups and link validation.
```

Create this empty file (with only the header comments) on first run. It is
populated incrementally as briefs are written. If the vault already has
concept pages but no registry (pre-existing vault), build the registry by
scanning `_concepts/*.md` for "## Seen In" entries before proceeding.

**Schema (v2 — enriched):** Each entry includes a `category` field sourced
from the concept page's frontmatter, plus a `projects` list:

```yaml
strategy-pattern:
  category: pattern
  projects:
    - expressjs--express
    - pallets--flask
```

---

## _concepts/_vault-state.yaml (Vault State)

```yaml
# _concepts/_vault-state.yaml
# Auto-maintained by codebase-educator and educator-audit.
# Tracks per-project metadata for incremental operations.
vault-schema-version: 2
last-global-audit: null
projects: {}
```

Create this empty file on first run. It is updated by:
- **codebase-educator Phase 4** — sets `last-educator-run` and `concept-count`
  for the project after committing
- **educator-audit Phase 6** — sets `last-audit` for audited projects and
  `last-global-audit` for vault-wide audits

Per-project entry schema:

```yaml
projects:
  expressjs--express:
    last-educator-run: "2026-03-30T14:00:00-04:00"  # set by educator Phase 4
    last-audit: "2026-03-30T15:00:00-04:00"          # set by audit Phase 6
    concept-count: 18                                 # concepts linked to this project
```

---

## _concepts/_connections.yaml (Connection Index)

```yaml
# _concepts/_connections.yaml
# Auto-maintained by codebase-educator. Do not edit manually.
# Pre-computed project-to-project concept overlaps.
```

Create this empty file on first run. It is updated incrementally by
codebase-educator Phase 3 when a new brief is added: compute the new
project's overlaps with all existing projects and append entries for both
directions.

Entry schema:

```yaml
expressjs--express:
  pallets--flask: [middleware, layered-architecture, decorator-based-routing]
  withastro--astro: [plugin-architecture, chain-of-responsibility]
```

Keyed by project; value is a map of connected projects to their shared
concept list (sorted alphabetically). Both directions are stored so lookups
are O(1) — if A connects to B, both `A.B` and `B.A` exist.

---

## _index.md (Map of Content)

```markdown
# Educator Briefs

A growing knowledge base of codebase analyses. Each analysis deconstructs a
project's architecture, technology choices, design patterns, and lessons learned.

Open this folder as an Obsidian vault to see the concept graph — connections
between projects through shared patterns, philosophies, and architectural decisions.

## Analyzed Sources

| Source | Type | Date | Summary |
|--------|------|------|---------|
<!-- New entries added here by the codebase-educator skill -->

## Concepts

Browse `_concepts/` for cross-project pattern and principle pages, or use
Obsidian's graph view to explore connections visually.

## How to Use This Vault

1. **Start with a project overview** — each project folder has a `_<project-name>_overview.md`
   that links to all sections (the `_` prefix keeps it sorted to the top)
2. **Follow concept links** — `[[wikilinks]]` connect project observations to
   general principles in `_concepts/`
3. **Use the graph view** — Obsidian shows how projects connect through shared
   concepts. Clusters reveal your recurring architectural themes.
4. **Read learning paths** — each project has a `learning-path.md` with an
   ordered reading list through the source code
```

---

## Obsidian Settings

On first run, also create `~/.claude/educator-briefs/.obsidian/` with a minimal
config if it doesn't exist:

### .obsidian/app.json

```json
{
  "showLineNumber": true,
  "strictLineBreaks": false,
  "readableLineLength": true
}
```

### .obsidian/graph.json

```json
{
  "collapse-filter": false,
  "search": "",
  "showTags": false,
  "showAttachments": false,
  "hideUnresolved": false,
  "showOrphans": true,
  "collapse-color-groups": false,
  "colorGroups": [
    {
      "query": "path:_concepts",
      "color": { "a": 1, "rgb": 5431178 }
    }
  ],
  "collapse-display": false,
  "showArrow": true,
  "textFadeMultiplier": 0,
  "nodeSizeMultiplier": 1,
  "lineSizeMultiplier": 1,
  "collapse-forces": true,
  "centerStrength": 0.5,
  "repelStrength": 10,
  "linkStrength": 1,
  "linkDistance": 250
}
```

This gives concept pages a distinct color in the graph view so they visually
stand out as the connecting nodes between projects.
