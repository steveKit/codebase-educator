---
name: codebase-educator
description: Analyze a codebase and produce an educational brief covering architecture, technology choices, design patterns, and gaps. Use when the user asks to "educate me on this codebase", "analyze this project for learning", "explain the architecture", "what can I learn from this code", or "codebase educator".
argument-hint: "[source...] -- one or more: local path, GitHub URL, website URL, npm:package, pypi:package, or omit for current project"
disable-model-invocation: true
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, WebFetch, Agent]
---

# Codebase Educator

Analyze a source and produce a structured educational brief as an Obsidian-compatible
vault. The vault lives on the Windows filesystem at `/mnt/c/Users/horse/Obsidian/educator-briefs/`
and is symlinked to `~/.claude/educator-briefs/`. Always use the symlink path for
file operations -- the safety hook allowlists the resolved path.

**Arguments**: $ARGUMENTS

---

## Architecture: State Machine + Registers

This skill uses a **state machine** for restartability and **disk registers** for
context efficiency. Registers are YAML files in `/tmp/educator-<name>/` that
replace implicit context-window memory with explicit, shareable data structures.

### State Machine

```
INIT -> GATHERING -> GATHERED -> ASSESSED -> WRITING -> SECTIONS_DONE
-> SWEPT -> CONCEPTS_DONE -> COMMITTED -> COMPLETE
```

Transition rules and the full schema are in `references/registers/state.md`.
Write the state file after every transition. On resume, read the state file
and skip completed phases.

### Registers

| Register | Written by | Purpose | Schema |
|---|---|---|---|
| `state.yaml` | Orchestrator | Workflow progress + restartability | `references/registers/state.md` |
| `gather.yaml` | Phase 1 | Structured codebase data | `references/registers/gather.md` |
| `url-index.yaml` | Phase 1 | Technology link lookup table | `references/registers/url-index.md` |
| `quality.yaml` | Phase 1.5 | Quality assessment + tone guidance | `references/registers/quality.md` |
| `sections.yaml` | Phase 2 | Per-section metadata (concepts, URLs, counts) | `references/registers/sections-manifest.md` |

All registers live in `/tmp/educator-<name>/`. Read register schemas on first
use, not upfront -- only load the schema you need for the current phase.

### Context Loading Rules

**Load only what the current phase needs:**

| Phase | Load these references | Load these registers |
|---|---|---|
| 1 (Gather) | nothing extra | -- (creating them) |
| 1.5 (Quality) | `quality-assessment.md` | `gather.yaml` |
| 2 (Write) | `sections/_shared.md` + the ONE section template | `gather.yaml`, `url-index.yaml`, `quality.yaml` |
| 2.5 (Sweep) | nothing extra | `sections.yaml` |
| 3 (Concepts) | `concept-template.md` | `sections.yaml`, registry on disk |
| 4-5 (Commit/Report) | nothing extra | `state.yaml`, `sections.yaml` |

**Never load all 13 section templates at once.** Each section writer loads
`references/sections/_shared.md` (shared rules) + its own template file only.

---

## Source Detection

`$ARGUMENTS` may contain **one or more sources**, space-separated. Parse each
token using:

| Input | Type | Method |
|---|---|---|
| No argument | Local project | Analyze current working directory |
| `/path`, `./path`, `~/path` | Local path | Analyze directory at that path |
| `https://github.com/...` | GitHub repo | `git clone --depth 1` to `/tmp/educator-<name>` |
| `https://...` (non-GitHub) | Website | Discover repo first (see Website Sources below), fall back to external observation |
| `npm:<package>` | npm package | `npm pack` to `/tmp/educator-<name>`, extract |
| `pypi:<package>` | PyPI package | `pip download --no-deps --no-binary :all:` to `/tmp/educator-<name>`, extract |

### Source URL Resolution

Resolve the base URL once during source detection:

| Source type | Base URL | File link pattern |
|---|---|---|
| GitHub repo | Input URL | `<base>/blob/<branch>/<filepath>` |
| Local with GitHub remote | `git remote get-url origin` converted | Same blob pattern |
| npm/PyPI with `repository` | From package manifest | Same if GitHub |
| Local, no remote | `null` | Relative paths only |

### Multi-Source Batching

When multiple sources are provided:

| Phase | Scope |
|---|---|
| 1, 1.5, 2, 2.5 | **Per source** sequentially |
| 3 (Concepts) | **Once** across all sources |
| 4 (Commit) | **Once** -- single branch, single commit |
| 5 (Report) | **Once** -- combined report |

Clean up each source's `/tmp/` directory after its Phase 2 completes.
Build a shared URL index across sources -- check existing entries before
making new lookups.

---

## Phase 1: Gather

**State transition:** `INIT -> GATHERING -> GATHERED`

1. **Resolve source** -- detect type, acquire code if needed
2. **Scan structure** -- `ls` top-level, read key files:
   - Entry points (main, index, app, server files)
   - Config files (package.json, Cargo.toml, go.mod, pyproject.toml, etc.)
   - README, CLAUDE.md, ARCHITECTURE.md, docs/ if they exist
   - CI/CD config (.github/workflows, Dockerfile, docker-compose)
   - Test directories and config
3. **Map dependency graph** -- read imports in key files, trace module structure
4. **Sample depth** -- read files from different codebase areas:

   | Project size | Sample target | Strategy |
   |---|---|---|
   | Small (<20 files) | Read most files | Near-complete coverage |
   | Medium (20-100) | 8-15 files | Cover every layer/module |
   | Large (100+) | 15-25 files | Glob to discover, sample each subsystem |

   Batch Read calls. Prioritize: hub modules, architectural boundaries,
   unusual files. For each file, write a **structured summary** (not raw
   content) into the gather register, noting key snippets by line range.

5. **Read test files** -- 2-3 real test files for core logic (not trivial utils)
6. **Check history** (if git) -- `git log --oneline -20` for activity;
   `git log --diff-filter=A --name-only --format="" | head -30` for file creation order
7. **Build URL index** -- For every significant technology, resolve:
   - Official docs URL (construct from known patterns for well-known tech)
   - Registry URL (npm/PyPI/crates.io/pkg.go.dev patterns)
   - Repo URL (from package manifest or known GitHub paths)

   **WebFetch budget:** Only verify URLs when docs site isn't obvious from
   the name or registry metadata. Never WebFetch well-known tech URLs.
   When uncertain, link to registry page.

   **Minimum coverage:** language, framework, database, major libraries
   (3+ imports), build tools, test framework.

8. **Write registers:**
   - Write `gather.yaml` following `references/registers/gather.md` schema
   - Write `url-index.yaml` following `references/registers/url-index.md` schema
   - Update `state.yaml`: state -> `GATHERED`

---

## Phase 1.5: Quality Assessment

**State transition:** `GATHERED -> ASSESSED`

Load `references/quality-assessment.md` for the full rubric.

1. Read `gather.yaml` for codebase observations
2. Evaluate all 6 quality signals (engineering maturity, maintenance health,
   testing discipline, documentation intent, security awareness, dependency hygiene)
3. Assign overall rating: **exemplary**, **solid**, **mixed**, or **cautionary**
4. Pre-compute per-section quality notes
5. Write `quality.yaml` following `references/registers/quality.md` schema
6. Update `state.yaml`: state -> `ASSESSED`

---

## Phase 2: Write Sections

**State transition:** `ASSESSED -> WRITING -> SECTIONS_DONE`

### Vault Bootstrap (first-ever run only)

If `~/.claude/educator-briefs/_index.md` doesn't exist, create the vault
structure. See `references/vault-bootstrap.md` for templates:
- `_index.md` (Map of Content)
- `_concepts/` directory
- `_concepts/_registry.yaml` (empty with header comments)

If the project subfolder already exists, ask the user whether to overwrite
or create a timestamped version.

### Section Writing Protocol

For each section, the writer:

1. Loads `references/sections/_shared.md` (shared rules)
2. Loads `references/sections/<section>.md` (that section's template only)
3. Reads registers: `gather.yaml`, `url-index.yaml`, `quality.yaml`
4. Does **targeted reads** of 2-5 specific files/line ranges from the
   source for code examples (guided by `sampled-files` in gather register)
5. Writes the section file to `~/.claude/educator-briefs/<project-name>/`
6. Appends its entry to `sections.yaml` (concepts linked, URLs used, counts)
7. Updates `state.yaml` section status: `done`

### Write Order and Parallelism

**Track A** (implementation-heavy -- these need more source file reads):
1. `architecture.md`
2. `design-patterns.md`
3. `key-decisions.md`
4. `testing-strategy.md`
5. `gaps-vulnerabilities.md`

**Track B** (analytical -- these draw more from registers than raw source):
1. `technology-choices.md` (can start after `architecture.md` completes)
2. `dependencies.md`
3. `evolution.md`
4. `if-starting-over.md`
5. `learning-path.md`
6. `glossary.md`

**Sequencing rule:** `architecture.md` is always written first (it
establishes structural vocabulary). After that, both tracks can proceed.
The following are always last, in order:
- `resources.md` (collects from all other sections -- read `sections.yaml`)
- `_<project-name>_overview.md` (summarizes everything)

**Parallel dispatch (when using Agent tool):**

If dispatching agents for Track A and Track B:
- Each agent gets: its section templates + all three registers
- No file overlap -- each agent writes different section files
- Each agent appends to `sections.yaml` (append-only, no conflicts)
- Instruct agents to use `Bash` with heredoc for new file writes
  (background agents may get `Write` denied for vault paths)
- The orchestrator writes `resources.md` and `overview.md` last

If running single-threaded, write in this order:
`architecture` -> `technology-choices` -> `design-patterns` ->
`key-decisions` -> `dependencies` -> `evolution` -> `testing-strategy` ->
`gaps-vulnerabilities` -> `if-starting-over` -> `learning-path` ->
`glossary` -> `resources` -> `overview`

### Mermaid Validation

After writing all sections, validate every Mermaid diagram:
- Direction declaration present (`graph TD`, `graph LR`, etc.)
- Every `subgraph` has a matching `end`
- No special characters in labels without wrapping (`A["Label (parens)"]`)
- No duplicate node IDs within a diagram
See `references/diagram-guide.md` for syntax patterns.

### Inline Link Checklist

After all sections are written, verify:
1. Every technology hyperlinked on first mention per section
2. `technology-choices.md` stack table has clickable links in every row
3. `dependencies.md` has registry + docs links for every dependency
4. `glossary.md` has docs links for framework-specific jargon
5. `gaps-vulnerabilities.md` links to guides for addressing each gap
6. `resources.md` contains every unique URL from all sections

Update `state.yaml`: state -> `SECTIONS_DONE`

---

## Phase 2.5: Concept Sweep

**State transition:** `SECTIONS_DONE -> SWEPT`

Scan for missed concept wikilink opportunities.

1. Read `sections.yaml` to see what's already linked
2. Grep section files for `##` and `###` headers
3. Apply the **transferability test** to each header naming a technique/pattern:

   > Would this concept page make sense with zero projects in "Seen In"?
   > Can you write "How It Works" and "Trade-Offs" without referencing any project?

   Yes -> it's a concept, add `[[concept-name]]` wikilink near the header.
   No -> it's a project detail, leave it.

4. Priority targets: named patterns in design-patterns.md, architectural styles
   in architecture.md, named practices in testing-strategy.md, anti-patterns,
   missing patterns
5. Add missing wikilinks inline. Update `sections.yaml` with new concepts.
6. Typical yield: 2-5 new concepts. If 15+, threshold is too low.
7. Update `state.yaml`: state -> `SWEPT`, concept-sweep-done -> true

---

## Phase 3: Concepts & Index

**State transition:** `SWEPT -> CONCEPTS_DONE`

Load `references/concept-template.md` for the concept page template.

### Registry

Uses `_concepts/_registry.yaml` -- a YAML index mapping concept names to
projects. If the registry doesn't exist (older vault), build it once by
scanning `_concepts/*.md` for "## Seen In" entries.

### Steps

1. **Load registry** into memory
2. **Collect concept list** from `sections.yaml` -- union of all
   `concepts-linked` across sections. Deduplicate.
3. **Process concepts** -- for each:
   - **Normalize to kebab-case:** lowercase, hyphens only. This is the
     filename AND registry key AND wikilink target.
   - Check registry (not filesystem) for existence
   - **New concept:** Create page from `references/concept-template.md`,
     add to registry with this project as first entry
   - **Existing concept:** Append backlink under "## Seen In", add project
     to registry list

   **Backlink format:** `[[<project>/_<project>_overview|<project>]]`
   (bare `[[project]]` resolves to nothing -- it's a folder).

   **Batch writes:** Group new concepts and write in quick succession.
   Batch Edit calls for existing concept backlinks.

4. **Cross-project connections** -- Use registry to find shared concepts:
   - Filter entries where the list contains both new project + others
   - Read "## Seen In" in shared concept pages for usage descriptions
   - Write "## Cross-Project Connections" in new project's overview
   - **Update each connected project's overview** with reciprocal entry
   - Format:
     ```
     Concepts shared with [[other/_other_overview|Display Name]]:
     - **Concept** -- how this project uses it; how the other uses it.
     ```

5. **Write registry** back to disk
6. **Update `_index.md`** -- add project row, update Concepts by Category
7. **Quick link checks:**
   - All wikilinks are lowercase-with-hyphens
   - Prefer alias links to broad concepts over near-duplicates
   - No speculative links -- only wikilink concepts actually discussed
8. Update `state.yaml`: state -> `CONCEPTS_DONE`, populate concepts-created/updated/connections

---

## Phase 4: Commit & Push

**State transition:** `CONCEPTS_DONE -> COMMITTED`

1. `cd ~/.claude/educator-briefs`
2. Create branch: `git checkout -b brief/<project-name>`
   (multi-source: `brief/batch-YYYY-MM-DD`)
3. Stage files: `git add <project>/ _concepts/ _index.md`
   Include any existing project overviews updated with reciprocal connections.
4. Commit: `git commit -m "feat(<project>): add educational brief"`
5. Push: `git push -u origin brief/<project-name>`
6. Create PR: `gh pr create --title "feat(<project>): add educational brief" --body "..." --fill`
7. Merge: `gh pr merge --squash --delete-branch`
8. Return: `git checkout main && git pull`
9. Update `state.yaml`: state -> `COMMITTED`, populate branch/sha/pr

---

## Phase 5: Report

**State transition:** `COMMITTED -> COMPLETE`

Present to the user:
- Vault location (`~/.claude/educator-briefs/`)
- Per-source: brief summary (3-5 key findings)
- Total new concept pages created
- Cross-project connections discovered
- Suggestion to open as an Obsidian vault
- **Prompt to run validation:** "Run `/educator-audit <project-name>` to
  validate links, registry consistency, and section quality."

Update `state.yaml`: state -> `COMPLETE`

---

## Cleanup

- `rm -rf /tmp/educator-<name>` and **check exit code**
- If deletion fails: tell the user, provide the manual command, explain why
- Never leave cloned repos in `/tmp/` without the user knowing

---

## Website Sources

### Repo Discovery

Use one WebFetch call on the main URL. Look for GitHub/GitLab/Bitbucket links in
footer, header, meta tags. Try `https://github.com/<domain-parts>` for obvious cases.

### Repo Found -> Code Analysis

Clone as GitHub source. Run full analysis. Note discovery in overview:
`**Source:** [owner/repo](url) *(discovered via [domain](url))*`

### No Repo -> External Observation

Use WebFetch on main URL + 2-3 subpages. Analyze observable tech choices.
Skip `evolution.md` and `testing-strategy.md`. Mark as "external observation only."
Omit code examples and relevant files headers. Compensate with deeper analytical
prose, Mermaid diagrams, and external resource links.

---

## Project Subfolder Naming

- Local path -> directory name (e.g., `my-app`)
- GitHub -> `owner--repo` (e.g., `expressjs--express`)
- Website -> domain name (e.g., `stripe.com`)
- Package -> package name (e.g., `fastify`)

If subfolder exists, ask user: overwrite or timestamped version?

### Project Subfolder Layout

```
<project-name>/
+-- _<project-name>_overview.md
+-- architecture.md
+-- technology-choices.md
+-- design-patterns.md
+-- key-decisions.md
+-- gaps-vulnerabilities.md
+-- dependencies.md
+-- evolution.md
+-- testing-strategy.md
+-- if-starting-over.md
+-- learning-path.md
+-- glossary.md
+-- resources.md
```

---

## Resume Protocol

On invocation, before source detection:

1. Check for existing `/tmp/educator-*/state.yaml` files
2. If found and state is not `COMPLETE`, offer to resume:
   "Found in-progress analysis for `<source>` (state: `<state>`). Resume?"
3. On resume: read state file, load registers for current state, continue
4. If registers are missing, fall back to the previous state that creates them

---

## Guidelines

- **Teach, don't lecture.** Knowledgeable colleague, not textbook.
- **Honest assessment.** If the code is messy, say so with empathy.
- **No moralizing.** Focus on architectural observations, not style lint.
- **Transferable knowledge first.** Every observation teaches a reusable principle.
- **Difficulty calibration.** Tag sections honestly.
- **Living vault.** Concept pages accumulate cross-project evidence over time.
- **Filename is `_<project-name>_overview.md`** -- never `_overview.md` or `overview.md`.
- **Concept filenames must be kebab-case.** `dependency-injection.md` not
  `Dependency-Injection.md`. #1 cause of orphaned links.
- **One writer per file.** Never have main session AND an agent write the same file.
- **Background agents: use Bash for file writes.** `Write` may be denied for
  vault paths. Use `cat > "$file" << 'CONTENT'` as primary write method.
  `Edit` works for modifications to existing files.
