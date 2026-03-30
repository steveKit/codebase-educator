---
name: educator-audit
description: Audit the educator-briefs vault for link integrity, registry consistency, and section quality. Use when the user asks to "audit the vault", "check educator links", "validate briefs", "fix orphaned links", or "educator audit".
argument-hint: "[project-name] — audit one brief, or omit for vault-wide audit"
disable-model-invocation: false
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, WebFetch]
---

# Educator Audit

Validate and repair the educator-briefs vault. Catches what codebase-educator's
creation process misses: orphaned wikilinks, registry drift, broken external
URLs, missing backlinks, and section quality gaps.

**Vault path:** `~/.claude/educator-briefs/` (symlink — use this, not `/mnt/`).

**Arguments**: $ARGUMENTS

## Modes

| Input | Scope | What's checked |
|---|---|---|
| No argument | **Vault-wide** | All projects, all concepts, full registry, `_index.md` |
| `<project-name>` | **Single brief** | That project's sections + its concept links + registry entries |

## Process

### Phase 1: Inventory

1. **Load registry** — Read `_concepts/_registry.yaml` into memory. If it
   doesn't exist, note this as a critical finding (codebase-educator should
   have created it).

2. **Discover projects** — Glob `~/.claude/educator-briefs/*/` to find all
   project subdirectories. Exclude `_concepts/`. If a `<project-name>` argument
   was provided, filter to just that project (error if it doesn't exist).

3. **Discover concept files** — Glob `~/.claude/educator-briefs/_concepts/*.md`
   to get the list of actual concept pages on disk.

4. **Read `_index.md`** — Parse the Projects table to know which projects are
   currently indexed.

### Phase 2: Link Integrity

Run these checks per project (or all projects in vault-wide mode):

#### 2a. Wikilink Resolution

For each project's section files (`*.md` in the project folder):

1. **Extract all wikilinks** — Grep for `\[\[([^\]]+)\]\]` patterns.
2. **Classify each link:**
   - `[[concept-name]]` or `[[concept-name|Display]]` → concept link
   - `[[project/section]]` or `[[project/section|Display]]` → cross-ref link
   - `[[project/_<project-name>_overview|display]]` → project link (used in concept backlinks)
3. **Validate concept links:**
   - Strip alias (`[[foo|Bar]]` → `foo`)
   - Check that `foo` exists as a key in the registry AND as a file
     `_concepts/foo.md` on disk
   - If in registry but no file → **critical** (registry corrupt)
   - If file exists but not in registry → **fixable** (add to registry)
   - If neither → **orphaned link** (fixable — see Phase 3)
4. **Validate cross-ref links:**
   - `[[project/section]]` → verify `section` is one of the known section
     filenames: `_<project-name>_overview`, `architecture`, `technology-choices`,
     `design-patterns`, `key-decisions`, `gaps-vulnerabilities`,
     `dependencies`, `evolution`, `testing-strategy`, `if-starting-over`,
     `learning-path`, `glossary`, `resources`
   - Verify the target project folder exists on disk
5. **Check casing (kebab-case)** — All concept link targets AND filenames must
   be `lowercase-with-hyphens` (kebab-case). Flag and auto-fix any that contain:
   - Uppercase characters (`Dependency-Injection` → `dependency-injection`)
   - Underscores (`cooperative_multitasking` → `cooperative-multitasking`)
   - Spaces or other non-hyphen separators
   This is the most common cause of orphaned concept links — the wikilink
   target doesn't match the actual filename.

#### 2b. External URL Validation

For each project's section files:

1. **Extract all markdown links** — Grep for `\[([^\]]+)\]\(([^)]+)\)` patterns.
2. **Categorize URLs:**
   - **Registry URLs** (npm, PyPI, crates.io, pkg.go.dev, GitHub) — validate
     by pattern only. These follow predictable formats; if the pattern is
     well-formed, the URL is almost certainly valid. Check:
     - `npmjs.com/package/<name>` — name is non-empty
     - `pypi.org/project/<name>/` — name is non-empty
     - `crates.io/crates/<name>` — name is non-empty
     - `github.com/<owner>/<repo>` — both segments present
   - **Known-stable docs** — Major technology docs sites (MDN, React, Express,
     TypeScript, Rust, Python, Go, etc.) are pattern-validated only. If the URL
     follows the standard docs site structure, accept it.
   - **Other URLs** — WebFetch with a budget of **10 calls per audit run**.
     Prioritize URLs that appear in multiple sections or look unusual. For each,
     check that the page loads (non-404). Record failures.
3. **Flag issues:**
   - Malformed URLs (missing scheme, broken encoding)
   - Empty link text `[](url)` or empty URL `[text]()`
   - Duplicate URLs pointing to different link texts (inconsistency, not error)

#### 2c. `resources.md` Completeness

For each project:

1. **Collect all external URLs** from every section file except `resources.md`
2. **Read `resources.md`** and extract its URLs
3. **Diff** — any URL present in sections but missing from `resources.md` is
   a gap. Report it.

### Phase 3: Registry & Index Consistency

#### 3a. Registry ↔ Filesystem Sync

Compare the registry keys against actual `_concepts/*.md` files:

| Condition | Severity | Auto-fix |
|---|---|---|
| Key in registry, no `.md` file | Critical | Create concept page from registry context |
| `.md` file exists, no registry key | Fixable | Add key to registry, scan "Seen In" for project list |
| Registry lists a project that has no folder | Warning | Remove stale project from registry entry |

#### 3b. Concept Backlink Completeness

For each concept in the registry:

1. Get the list of projects that wikilink to this concept (from Phase 2a data)
2. Get the list of projects in the concept's "Seen In" section
3. **Missing backlinks** — project uses the concept but isn't listed in "Seen In"
   → fixable
4. **Stale backlinks** — project listed in "Seen In" but doesn't actually
   wikilink the concept (or project folder doesn't exist) → fixable

#### 3c. `_index.md` Completeness

1. Compare discovered project folders against the `_index.md` Projects table
2. **Missing entries** — project folder exists but isn't in the table → fixable
3. **Stale entries** — project listed but folder doesn't exist → fixable
4. **Concepts by Category** — verify every concept page is listed under its
   category. Cross-reference the `category` field from each concept's YAML
   frontmatter.

#### 3d. Cross-Project Connections

For each project in scope:

1. Use the registry to find concepts shared with other projects
2. Read the project's `_<project-name>_overview.md` "Cross-Project Connections" section
3. **Missing connections** — shared concept exists but no connection block
   in either project's overview → fixable
4. **One-directional connections** — project A mentions B but B doesn't
   mention A → fixable

### Phase 4: Section Quality

Per-project quality checks. These are **reported, not auto-fixed** — fixing
requires regeneration, not mechanical repair.

#### 4a. Structure

For each section file:

| Check | Pass criteria |
|---|---|
| YAML frontmatter | Has `source`, `section`, `difficulty`, `quality-note` fields |
| Relevant files block | Present (except `_<project-name>_overview`, `if-starting-over`, `resources`) |
| "Why This Matters" callout | Present at end of section |
| Section file exists | All 13 expected files present in project folder |

#### 4b. Depth

| Section category | Min words | Min code examples |
|---|---|---|
| Implementation-heavy (`architecture`, `design-patterns`, `key-decisions`, `testing-strategy`, `gaps-vulnerabilities`) | 300 | 3 |
| Analytical (`technology-choices`, `dependencies`, `evolution`, `if-starting-over`) | 200 | 1 |
| Reference (`learning-path`, `glossary`, `resources`, `_<project-name>_overview`) | Complete coverage | 0 |

Count words (excluding YAML frontmatter and code blocks) and code fences per file.

#### 4c. Mermaid Diagrams

For each ` ```mermaid ` block:

- Has a direction declaration (`graph TD`, `graph LR`, `sequenceDiagram`, etc.)
- Every `subgraph` has a matching `end`
- Node IDs don't contain unescaped special characters
- No duplicate node IDs within a diagram

Report issues with file path and line number.

#### 4d. Inline Link Density

Check first-mention linking per section (spot check, not exhaustive):

- `technology-choices.md` — Stack Summary table should have links in every row
- `dependencies.md` — every dependency name should be a link on first mention
- `glossary.md` — framework-specific terms should link to docs

Report sections that appear under-linked.

### Phase 5: Auto-Fix

Apply fixes for all issues marked **fixable** in Phases 2-3. Group by type:

#### 5a-pre. Kebab-Case Normalization

Before processing orphaned links, fix casing issues (from Phase 2a step 5):

1. **Rename non-kebab-case concept files** — `git mv` the file to the
   normalized name (lowercase, hyphens only). Update `_registry.yaml` key.
2. **Fix wikilinks** — In every section file that references the old name,
   replace `[[Old-Name]]` with `[[old-name]]` (or `[[old-name|Old Name]]`
   if an alias is needed for display). Use `replace_all` for efficiency.
3. **Fix backlinks** — Update "Seen In" entries in concept pages that
   reference the renamed concept.

This step resolves many "orphaned" links that are really just casing mismatches.

#### 5a. Orphaned Wikilinks

For each orphaned concept link (remaining after 5a-pre):

1. Check if a similar concept already exists in the registry (fuzzy match —
   e.g., `middleware-pattern` vs `middleware`). If so, fix the wikilink to
   use an alias: `[[middleware|middleware-pattern]]`
2. If no similar concept exists, create the concept page using the concept
   template from `codebase-educator/references/concept-template.md`. Fill in:
   - `concept`: infer from the link text
   - `category`: infer from the section context where the link appears
   - `difficulty`: infer from the section's difficulty tag
   - "Seen In" entry for the current project
   - Mark the page with `<!-- auto-generated by educator-audit -->` after
     the YAML frontmatter so it can be identified for later enrichment
3. Add the concept to the registry

#### 5b. Registry Repairs

- Add missing keys (from files without registry entries)
- Remove stale project references
- Sort registry alphabetically by key

#### 5c. Backlink Repairs

- Append missing "Seen In" entries to concept pages
- Remove stale "Seen In" entries for deleted projects

#### 5d. Index Repairs

- Add missing project rows to `_index.md` Projects table (read `_<project-name>_overview.md`
  for the summary and date)
- Remove stale project rows
- Add missing concepts to the "Concepts by Category" section

#### 5e. Cross-Project Connection Repairs

- Add missing reciprocal connection entries to `_<project-name>_overview.md` files
- Format matches the codebase-educator convention:
  ```
  Concepts shared with [[<other-project>/_<other-project>_overview|<Display Name>]]:
  - **<Concept>** — <how this project uses it>; <how the other uses it>.
  ```
- For auto-generated entries, use the "Seen In" descriptions from concept pages
  as the usage summary. Mark with `<!-- auto-generated by educator-audit -->`

#### 5f. `resources.md` Gap Fill

- Append missing URLs to the appropriate table in `resources.md`
- Use the link text and URL from the section where the link was found
- Add to the most appropriate category (Core Stack, Dependencies, Pattern
  References, or Further Reading)

### Phase 6: Commit & Push

If any files were modified:

1. `cd ~/.claude/educator-briefs`
2. `git checkout -b audit/<project-name>` (or `audit/vault-YYYY-MM-DD` for
   vault-wide)
3. Stage modified files: `git add -A`
4. Show `git diff --cached --stat` to the user
5. **Wait for user approval before committing.** Present the summary of
   changes and ask "Commit these fixes?"
6. On approval:
   - `git commit -m "fix(<scope>): audit repairs — <summary>"`
   - `git push -u origin audit/<scope>`
   - `gh pr create --title "fix(<scope>): audit repairs" --body "<details>" --fill`
   - `gh pr merge --squash --delete-branch`
   - `git checkout main && git pull`

### Phase 7: Report

Present a structured summary:

```
## Educator Audit Report — <scope>

### Link Integrity
- Wikilinks checked: N
- Orphaned links found: N (N fixed)
- External URLs checked: N
- Broken URLs found: N
- resources.md gaps: N (N filled)

### Registry & Index
- Concepts in registry: N
- Concept files on disk: N
- Registry mismatches fixed: N
- Missing backlinks added: N
- _index.md entries added/removed: N
- Cross-project connections added: N

### Section Quality
- Sections checked: N
- Missing frontmatter: N (list files)
- Below depth minimum: N (list files)
- Mermaid issues: N (list files + line numbers)
- Under-linked sections: N (list files)

### Summary
<one-paragraph overall assessment>
```

For vault-wide audits, break down findings per project before the totals.

## Guidelines

- **Fix what's mechanical, report what's creative.** Orphaned links, registry
  sync, and missing backlinks can be fixed automatically. Thin sections and
  missing code examples require regeneration — just report them.
- **Never delete a concept page.** Even if no project currently links to it,
  it may have been delinked by mistake. Flag it as "unreferenced" instead.
- **Preserve existing content.** When appending backlinks or connection entries,
  never modify existing text in the file — only append.
- **Registry is the source of truth.** When registry and filesystem disagree,
  investigate both before deciding which to fix. The registry is usually right
  for concept-to-project mappings; the filesystem is usually right for
  concept existence.
- **WebFetch budget is firm.** 10 calls max per audit run. Prioritize URLs
  that look suspicious or appear in multiple briefs. Pattern-validated URLs
  (registries, major docs sites) don't count against the budget.
- **Auto-generated markers.** Always mark auto-created content with
  `<!-- auto-generated by educator-audit -->` so codebase-educator can
  enrich it on a future analysis pass.
- **Idempotent.** Running the audit twice in a row with no changes between
  runs should produce zero findings on the second run. If it doesn't,
  the audit has a bug.
- **Ground every finding in evidence.** Every issue reported must
  reference a specific file and the exact content that triggered it.
  If a check is inconclusive (e.g., a URL cannot be verified within
  the WebFetch budget), report it as "unverified" rather than
  assuming pass or fail.
