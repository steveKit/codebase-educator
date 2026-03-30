---
name: educator-audit
description: Audit the educator-briefs vault for link integrity, registry consistency, and section quality. Use when the user asks to "audit the vault", "check educator links", "validate briefs", "fix orphaned links", or "educator audit".
argument-hint: "[project-name] [--since <duration|date>] [--full] — audit one brief, scope by recency, or force full vault-wide"
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
| No argument | **Smart vault-wide** | Projects modified since their `last-educator-run` in vault-state + vault-wide structural checks |
| `<project-name>` | **Single brief** | That project's sections + its concept links + registry entries (always full, ignores `--since`) |
| `--since <duration\|date>` | **Windowed vault-wide** | Only projects with git changes since the given time. Duration: `1w`, `2w`, `1m`. Date: `2026-03-15`. |
| `--full` | **Full vault-wide** | All projects, all concepts, full registry, `_index.md` (original behavior) |

### Smart Default Behavior

When invoked with no arguments and no flags:

1. Read `_concepts/_vault-state.yaml`
2. For each project, compare `last-educator-run` against `last-audit`
3. **Projects to audit:** those where `last-audit` is `null` OR `last-audit < last-educator-run`
4. Always include vault-wide structural checks (Phase 3a, 3c, 3d) regardless of windowing
5. If vault-state is missing or empty, fall back to `--full` behavior

## Process

### Phase 1: Inventory

1. **Load registry** — Read `_concepts/_registry.yaml` into memory (v2 enriched
   format with `category` + `projects` per entry). If it doesn't exist, note
   this as a critical finding. If it uses v1 flat format, note as fixable
   (migration needed).

2. **Load vault-state** — Read `_concepts/_vault-state.yaml`. If missing,
   note as fixable (will be created in Phase 5). Use it to determine which
   projects are in scope (see Modes above).

3. **Load connections** — Read `_concepts/_connections.yaml`. If missing,
   note as fixable (will be created in Phase 5).

4. **Discover projects** — Glob `~/.claude/educator-briefs/*/` to find all
   project subdirectories. Exclude `_concepts/`. If a `<project-name>` argument
   was provided, filter to just that project (error if it doesn't exist).
   If using smart default or `--since`, filter to in-scope projects for
   Phases 2 and 4 (Phases 3a, 3c, 3d remain vault-wide).

5. **Discover concept files** — Glob `~/.claude/educator-briefs/_concepts/*.md`
   to get the list of actual concept pages on disk.

6. **Read `_index.md`** — Parse the Projects table to know which projects are
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

**Phase 3 checks are always vault-wide**, even in windowed mode. These are
cheap structural checks and must catch drift regardless of which projects
were recently modified.

#### 3a. Registry ↔ Filesystem Sync

Compare the registry keys against actual `_concepts/*.md` files:

| Condition | Severity | Auto-fix |
|---|---|---|
| Key in registry, no `.md` file | Critical | Create concept page from registry context |
| `.md` file exists, no registry key | Fixable | Add key to registry with `category` from frontmatter, scan "Seen In" for project list |
| Registry lists a project that has no folder | Warning | Remove stale project from registry entry |
| Registry uses v1 flat format (no `category`) | Fixable | Migrate: read each concept's frontmatter, rewrite as v2 |
| Registry `category` doesn't match concept frontmatter | Fixable | Update registry to match frontmatter (frontmatter is authoritative) |

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
   category. Use the registry's `category` field (faster than reading every
   concept page's frontmatter). If a concept's registry category is `unknown`,
   read the frontmatter and fix the registry entry.

#### 3d. Cross-Project Connections

For each project in scope:

1. **Read `_connections.yaml`** for pre-computed overlaps (much faster than
   scanning the full registry). If missing, fall back to computing from registry.
2. Read the project's `_<project-name>_overview.md` "Cross-Project Connections" section
3. **Missing connections** — `_connections.yaml` shows overlap but no connection
   block in either project's overview → fixable
4. **One-directional connections** — project A mentions B but B doesn't
   mention A → fixable
5. **Stale connections** — connection referenced in overview but not in
   `_connections.yaml` or registry → warning (may indicate concept was delinked)
6. **Connections index drift** — if connections were computed from registry
   (fallback), compare against `_connections.yaml` and report discrepancies → fixable

#### 3e. Vault-State Consistency

1. Compare `_vault-state.yaml` projects against discovered project folders
2. **Missing entries** — project folder exists but not in vault-state → fixable
   (populate from git log)
3. **Stale entries** — project in vault-state but folder doesn't exist → fixable
   (remove entry)
4. **Concept count drift** — `concept-count` doesn't match actual registry
   count for that project → fixable

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

- Add missing keys (from files without registry entries) — include `category`
  from concept frontmatter in v2 format
- Migrate v1 flat entries to v2 enriched format (add `category`)
- Fix category mismatches (frontmatter is authoritative)
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

#### 5g. Connections Index Repairs

- If `_connections.yaml` is missing, rebuild from registry (compute all
  pairwise project overlaps)
- If `_connections.yaml` exists but has stale/missing entries, update
  incrementally — add missing pairs, remove stale ones
- Both directions must be present (A→B and B→A)

#### 5h. Vault-State Repairs

- If `_vault-state.yaml` is missing, create it: populate `last-educator-run`
  from `git log` per project, set `last-audit` to null, compute
  `concept-count` from registry
- Fix stale entries (projects that no longer exist)
- Add missing entries (projects that exist but aren't tracked)
- Fix concept-count drift

### Phase 6: Commit & Push

If any files were modified:

1. `cd ~/.claude/educator-briefs`
2. **Update `_vault-state.yaml`:**
   - For each audited project, set `last-audit` to current ISO timestamp
   - For vault-wide audits, also set `last-global-audit`
3. `git checkout -b audit/<project-name>` (or `audit/vault-YYYY-MM-DD` for
   vault-wide)
4. Stage modified files: `git add -A`
5. Show `git diff --cached --stat` to the user
6. **Wait for user approval before committing.** Present the summary of
   changes and ask "Commit these fixes?"
7. On approval:
   - `git commit -m "fix(<scope>): audit repairs — <summary>"`
   - `git push -u origin audit/<scope>`
   - `gh pr create --title "fix(<scope>): audit repairs" --body "<details>" --fill`
   - `gh pr merge --squash --delete-branch`
   - `git checkout main && git pull`

### Phase 7: Report

Present a structured summary:

```
## Educator Audit Report — <scope>

### Scope
- Mode: <smart default | --since <value> | --full | single project>
- Projects audited: N of M total
- Reason: <"modified since last audit" | "all" | "user-specified">

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
- Registry v1→v2 migrations: N
- Missing backlinks added: N
- _index.md entries added/removed: N
- Cross-project connections added: N
- Connection index entries added: N

### Vault State
- Projects tracked: N
- Entries added/removed: N
- Concept count fixes: N
- Last audit timestamps updated: N

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
- **Clean up educator `/tmp/` after audit.** The codebase-educator skill
  defers `/tmp/educator-<name>/` cleanup until after audit. If you find
  a `/tmp/educator-*/` directory for the project you just audited,
  clean it up: `rm -rf /tmp/educator-<name>` and check exit code.
  This completes the educator→audit→cleanup lifecycle.
- **Vault-state is bookkeeping, not gating.** Missing or stale vault-state
  entries should be fixed silently. Never refuse to audit a project because
  its vault-state entry is missing — create it.
