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
