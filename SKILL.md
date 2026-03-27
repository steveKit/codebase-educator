---
name: codebase-educator
description: Analyze a codebase and produce an educational brief covering architecture, technology choices, design patterns, and gaps. Use when the user asks to "educate me on this codebase", "analyze this project for learning", "explain the architecture", "what can I learn from this code", or "codebase educator".
argument-hint: "[source...] — one or more: local path, GitHub URL, website URL, npm:package, pypi:package, or omit for current project"
disable-model-invocation: true
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, WebFetch]
---

# Codebase Educator

Analyze a source and produce a structured educational brief as an Obsidian-compatible
vault. The vault lives on the Windows filesystem at `/mnt/c/Users/horse/Obsidian/educator-briefs/`
and is symlinked to `~/.claude/educator-briefs/`. Always use the symlink path for
file operations — the safety hook allowlists the resolved path.

**Arguments**: $ARGUMENTS

## Source Detection

`$ARGUMENTS` may contain **one or more sources**, space-separated. Parse each
token individually using the table below. If no argument is provided, the
single source is the current working directory.

| Input | Type | Method |
|---|---|---|
| No argument | Local project | Analyze current working directory |
| `/path`, `./path`, `~/path` | Local path | Analyze directory at that path |
| `https://github.com/...` | GitHub repo | Clone to `/tmp/educator-<name>`, analyze, clean up |
| `https://...` (non-GitHub) | Website | Try to discover source repo first (see below), fall back to external observation |
| `npm:<package>` | npm package | `npm pack <package>` to `/tmp/educator-<name>`, extract, analyze, clean up |
| `pypi:<package>` | PyPI package | `pip download --no-deps --no-binary :all:` to `/tmp/educator-<name>`, extract, analyze, clean up |

For GitHub repos: `git clone --depth 1 <url> /tmp/educator-<name>` (shallow clone is sufficient).

After analysis, always clean up any temp directories.

### Source URL Resolution

Many features (code example links, relevant file lists) need a base URL to link
back to source files. Resolve this once during source detection and reuse it:

| Source type | Base URL | File link pattern |
|---|---|---|
| GitHub repo | The input URL (e.g., `https://github.com/expressjs/express`) | `<base>/blob/main/<filepath>` (verify default branch with `git rev-parse --abbrev-ref HEAD`) |
| Local project with GitHub remote | Run `git remote get-url origin` and convert `git@github.com:owner/repo.git` → `https://github.com/owner/repo` | Same blob pattern |
| npm/PyPI with `repository` field | Read from package.json `repository.url` or pyproject.toml `[project.urls]` | Same blob pattern if GitHub |
| Local project, no remote | None — use relative file paths only | `<filepath>` (no link) |
| Website | N/A | N/A |

Store the resolved base URL (or `null`) for use in Phases 2-3. When no URL is
available, code examples and relevant file lists still reference paths — they
just aren't clickable links.

## Vault Structure

All output goes to `~/.claude/educator-briefs/`. This is an Obsidian vault.

On **first ever run**, create:
- `~/.claude/educator-briefs/_index.md` — Map of Content (MOC), using this template:

  ```markdown
  # Educator Briefs — Map of Content

  A growing vault of educational analyses. Open any project overview to start
  exploring.

  ## How to Use This Vault

  1. Pick a project from the table below and open its overview
  2. Follow `[[wikilinks]]` to concept pages for cross-project learning
  3. Use Obsidian's Graph View to see how concepts connect across projects
  4. Concepts grow richer with each new analysis — revisit them over time

  ## Projects

  | Project | Source Type | Date | Summary |
  |---|---|---|---|

  ## Concepts by Category

  *Maintained by the codebase-educator skill as analyses are added.*

  ### Architecture
  ### Patterns
  ### Philosophy
  ### Practice
  ### Principles
  ```

- `~/.claude/educator-briefs/_concepts/` — shared concept directory
- `~/.claude/educator-briefs/_concepts/_registry.yaml` — concept-to-project index (see Phase 3)

Each analysis creates a **project subfolder** named after the source:
- Local path → directory name (e.g., `my-app`)
- GitHub → `owner--repo` (e.g., `expressjs--express`)
- Website → domain name (e.g., `stripe.com`)
- Package → package name (e.g., `fastify`)

If the subfolder already exists, **ask the user** whether to overwrite or create
a timestamped version (e.g., `my-app-2026-03`).

### Project Subfolder Layout

```
<project-name>/
├── _overview.md             # Executive summary + wikilinks to all sections
├── architecture.md          # System structure, layers, data flow
├── technology-choices.md    # Stack decisions with rationale and trade-offs
├── design-patterns.md       # Patterns in use, quality of application
├── key-decisions.md         # Non-obvious choices that shaped the codebase
├── gaps-vulnerabilities.md  # Architectural weak points, missing patterns
├── dependencies.md          # Dependency philosophy and risk assessment
├── evolution.md             # How the codebase has evolved over time
├── testing-strategy.md      # What's tested, how, and what's missing
├── if-starting-over.md      # Lessons learned, what to do differently
├── learning-path.md         # Ordered reading list of files/modules
├── glossary.md              # Domain and technical terms defined in context
└── resources.md             # Consolidated links to official docs, repos, and guides
```

## Process

### Multi-Source Batching

When multiple sources are provided, run phases per-source then shared:

| Phase | Scope | What happens |
|---|---|---|
| Phase 1 (Gather) | **Per source** | Resolve, scan, build URL index for each source sequentially |
| Phase 1.5 (Quality) | **Per source** | Assess quality for each source |
| Phase 2 (Write) | **Per source** | Write all section files for each source |
| Phase 3 (Concepts) | **Once, across all sources** | Load registry once, process concepts from all sources in one pass, find cross-project connections (including between batch sources) |
| Phase 4 (Commit) | **Once** | Single branch, single commit containing all sources |
| Phase 5 (Report) | **Once** | Combined report covering all sources |

**Shared URL index:** If multiple sources use the same technology (e.g., two
Node.js projects both use Express), the URL is resolved once and reused. Build
each source's URL index incrementally, checking the running index before
making new lookups.

**Temp directory cleanup:** Clean up each source's `/tmp/educator-<name>`
after its Phase 2 completes — don't hold all clones in `/tmp/` simultaneously.

For a single source, this collapses to the normal sequential flow with no
overhead.

### Phase 1: Gather

1. **Resolve source** — detect type, acquire code if needed
2. **Scan structure** — `ls` top-level, read key files:
   - Entry points (main, index, app, server files)
   - Config files (package.json, Cargo.toml, go.mod, pyproject.toml, etc.)
   - README, CLAUDE.md, ARCHITECTURE.md, docs/ if they exist
   - CI/CD config (.github/workflows, Dockerfile, docker-compose)
   - Test directories and config
3. **Map the dependency graph** — read import/require statements in key files,
   trace the module structure, identify layers and boundaries
4. **Sample depth** — read files from different areas of the codebase to
   understand coding style, error handling, and patterns in practice.
   Scale sampling with project size:

   | Project size | Sample target | Strategy |
   |---|---|---|
   | Small (<20 source files) | Read most files | Near-complete coverage is feasible |
   | Medium (20-100 files) | 8-15 files | Cover every layer/module, not just entry points |
   | Large (100+ files) | 15-25 files | Use Glob to discover, then sample each subsystem |

   Use Glob to discover files by pattern (e.g., `**/*.ts`, `**/routes/**`)
   then batch Read calls rather than exploring one file at a time. Prioritize:
   files with the most imports (hub modules), files at architectural boundaries,
   and files that are unusual or unexpected for the stack.
5. **Read test files** — read 2-3 actual test files (not just test config).
   Choose tests for core business logic, not trivial utility tests. The
   testing-strategy section depends on seeing real test code — mocking
   approaches, assertion styles, what's tested vs. what's skipped.
6. **Check history clues** (if git repo) — `git log --oneline -20` for recent
   activity patterns; `git log --diff-filter=A --name-only --format="" | head -30`
   for file creation order (reveals evolution)
7. **Build technology URL index** — Before writing any sections, compile a lookup
   table of every significant technology in the stack. For each one, resolve:
   - **Official site / docs URL** — construct from known patterns (see below)
   - **Registry URL** — construct from the predictable patterns in "URL Construction"
   - **Repository URL** — GitHub/GitLab link (often in package.json `repository`,
     pyproject.toml `[project.urls]`, Cargo.toml `[package]`, or go.mod module path)

   **URL verification budget:** Most major technologies have stable, well-known
   URLs that don't need runtime verification. Only use WebFetch to verify a URL
   when the official docs site is not obvious from the technology name or registry
   metadata (e.g., a lesser-known library with no `homepage` field in its package
   manifest). For well-known technologies (major languages, popular frameworks,
   databases), construct the URL directly — don't spend a WebFetch call confirming
   that `https://www.typescriptlang.org/` still exists. When in doubt, link to
   the registry page — it's always valid and always has a link to the real docs.

   This index is the **single source of truth** for links throughout all sections.
   Every section that mentions a technology pulls from this index rather than
   constructing URLs ad hoc. Build it now so writing is fast and consistent.

   **Minimum coverage:** language runtime, framework, database/datastore, major
   libraries (anything imported in 3+ files), build tools, test framework. Skip
   trivial utilities used in one place.

### Phase 1.5: Quality Assessment

Before writing any sections, run the quality assessment from
`references/quality-assessment.md`:

1. Evaluate all 6 quality signals (engineering maturity, maintenance health,
   testing discipline, documentation intent, security awareness, dependency hygiene)
2. Assign an overall rating: **exemplary**, **solid**, **mixed**, or **cautionary**
3. This rating determines the framing tone for every section — see the reference
   doc for tone guidance and observation markers
4. For **mixed** and **cautionary** sources, every pattern observation must be
   explicitly tagged: *worth emulating*, *acceptable trade-off*, *cautionary
   example*, or *anti-pattern* — with the correct alternative explained
5. For concept page backlinks, include the quality context so readers never
   mistake a bad implementation for a good reference

The goal is not gatekeeping — bad code is highly educational. The goal is
**honest framing** so the reader always knows whether they're looking at
something to emulate or something to learn from by contrast.

### Phase 2: Analyze & Write

Write each section file using the templates in `references/section-guide.md`.

**Critical rules for writing sections:**
- Every section starts with a YAML-like properties block for Obsidian:
  ```
  ---
  source: <project-name>
  section: <section-name>
  difficulty: beginner | intermediate | advanced
  quality-note: <how the source quality rating affects this section>
  ---
  ```
- Every section ends with a **"Why This Matters"** callout explaining the
  general principle — what the reader learns that transfers to other projects
- Use `[[concept-name]]` wikilinks for every architectural concept, design
  pattern, or philosophy mentioned — these link to `_concepts/` pages
- Use `[[<project-name>/section-name]]` for cross-references within the project
- **Use Mermaid diagrams** (` ```mermaid `) for architecture visualization —
  Obsidian renders them natively as SVG, so they reflow on any screen size
  (desktop, tablet, mobile). See `references/diagram-guide.md` for syntax
  patterns and style rules. Never use ASCII box-drawing characters for
  structural diagrams — they break on narrow viewports and mobile
- Be honest about uncertainty: "This appears to be..." rather than asserting
  intent you can't verify from code alone
- **Embed resource links inline** — see "Resource Linking" below
- **Relevant files header** — After the YAML properties block, list the key
  source files for this section (see "Relevant Files" below). Omit for sections
  that don't map to specific files (`_overview.md`, `if-starting-over.md`,
  `resources.md`).
- **Code examples with source attribution** — Include short, illustrative code
  snippets (see "Code Examples" below). Every snippet links back to its source.

**Minimum depth expectations:**

Every section must be substantive enough to teach something the reader couldn't
learn from a 30-second glance at the repo. Use these floors:

| Section category | Min depth | Code examples |
|---|---|---|
| **Implementation-heavy** (architecture, design-patterns, key-decisions, testing-strategy, gaps-vulnerabilities) | 300+ words of analysis, multiple subsections | 3-5 code snippets grounding observations in real source |
| **Analytical** (technology-choices, dependencies, evolution, if-starting-over) | 200+ words, structured analysis with evidence | 1-3 code snippets where they illustrate a point |
| **Reference** (learning-path, glossary, resources, _overview) | Complete coverage of all items | N/A — these are indexes, not analysis |

If a section would be thin (e.g., no test files exist for testing-strategy),
**say so explicitly and explain what the absence reveals** rather than writing a
stub. A missing testing strategy is itself a substantive finding.

**Inline link checklist (run after writing all sections):**

For each section file, verify:
1. Every technology/framework/library/tool is hyperlinked on first mention
2. Links use the technology URL index from Phase 1 (no ad hoc URL construction)
3. `technology-choices.md` Stack Summary table has clickable links in every row
4. `dependencies.md` has registry + docs links for every dependency discussed
5. `glossary.md` has docs links for framework-specific jargon
6. `gaps-vulnerabilities.md` links to guides for addressing each gap

Then verify `resources.md` contains every unique URL from all sections.

Write sections in this order (each builds on the previous):
1. `architecture.md`
2. `technology-choices.md`
3. `design-patterns.md`
4. `key-decisions.md`
5. `dependencies.md`
6. `evolution.md`
7. `testing-strategy.md`
8. `gaps-vulnerabilities.md`
9. `if-starting-over.md`
10. `learning-path.md`
11. `glossary.md`
12. `resources.md` (consolidated link reference)
13. `_overview.md` (last — it summarizes everything above)

### Resource Linking

Every section should embed links to official documentation and dependency pages
where they are contextually relevant. Additionally, a consolidated `resources.md`
collects all links in one place.

#### Inline Links (embedded in each section)

**Every mention of a technology, framework, library, or tool should be a
hyperlink on first mention in each section.** Use the technology URL index
built in Phase 1 — this is why you built it. Subsequent mentions in the same
section can be plain text.

Per-section linking requirements:

- **`technology-choices.md`** — Stack Summary table must have clickable links
  for every technology (see section-guide.md). Decision Analysis paragraphs
  link to official docs on first mention of each technology.
- **`architecture.md`** — Link frameworks and runtimes where they define the
  architecture (e.g., link the framework when discussing its routing or
  middleware pipeline).
- **`design-patterns.md`** — Link to authoritative pattern references (e.g.,
  refactoring.guru, language-specific pattern guides) for patterns discussed.
  Link to framework docs when a pattern is framework-provided.
- **`dependencies.md`** — Every dependency in the analysis gets its registry
  link, repo link, and docs link (where they exist). This section should be
  the most link-dense in the brief.
- **`testing-strategy.md`** — Link to testing framework and assertion library docs.
- **`gaps-vulnerabilities.md`** — Link to relevant guides for addressing gaps
  (e.g., OWASP guides, caching strategy articles).
- **`learning-path.md`** — Link to prerequisite reading for advanced topics.
  Link to official "getting started" guides for technologies the reader needs
  to learn.
- **`glossary.md`** — Technical terms that are framework-specific jargon should
  link to the relevant docs page for that framework/library.

Format inline links as standard markdown: `[<Technology>](<url>)`.
Place them naturally in the text where a reader would want to dive deeper, not
dumped in a list at the end of a paragraph. Use the short form
(`[TechName](url)`) not the verbose form (`[TechName docs](url)`) when the
technology name alone is sufficient context.

#### URL Construction

For well-known registries, construct URLs directly — these follow predictable patterns:

| Registry | Pattern | Example |
|---|---|---|
| npm | `https://www.npmjs.com/package/<name>` | `https://www.npmjs.com/package/express` |
| PyPI | `https://pypi.org/project/<name>/` | `https://pypi.org/project/flask/` |
| crates.io | `https://crates.io/crates/<name>` | `https://crates.io/crates/serde` |
| pkg.go.dev | `https://pkg.go.dev/<module>` | `https://pkg.go.dev/net/http` |
| GitHub | `https://github.com/<owner>/<repo>` | `https://github.com/expressjs/express` |

For official documentation sites of well-known technologies, construct the URL
directly without verification. For lesser-known libraries where the docs URL
isn't obvious, check the `homepage` field in the package manifest or use one
WebFetch call to verify. If a docs URL is uncertain, link to the registry page
instead — a working link to the right package is better than a broken link to
the docs.

#### `resources.md` (consolidated)

After writing all sections, produce `resources.md` with every external link
collected and organized:

```markdown
# Resources

## Core Stack
| Technology | Docs | Repository | Registry |
|---|---|---|---|
| Express | [expressjs.com](https://expressjs.com) | [GitHub](https://github.com/expressjs/express) | [npm](https://www.npmjs.com/package/express) |

## Dependencies
| Package | Purpose | Docs | Registry |
|---|---|---|---|
| helmet | Security headers | [helmetjs.github.io](https://helmetjs.github.io) | [npm](https://www.npmjs.com/package/helmet) |

## Pattern References
| Pattern | Reference |
|---|---|
| Middleware | [Express middleware guide](https://expressjs.com/en/guide/using-middleware.html) |

## Further Reading
- [Topic] — [Link] — Why this is relevant to this codebase
```

Omit any column that would be empty for all rows in a table. Only include the
"Further Reading" section when there are genuinely useful supplementary resources
beyond docs and registries.

### Relevant Files

Each section (where applicable) starts with a **Relevant Files** block immediately
after the YAML properties. This gives the reader direct access to the source files
that informed the section.

**Format when a source URL is available (GitHub, etc.):**

```markdown
> **Relevant files:**
> [`<path>`](<base-url>/blob/<branch>/<path>) ·
> [`<path>`](<base-url>/blob/<branch>/<path>) ·
> [`<path>`](<base-url>/blob/<branch>/<path>)
```

**Format when no source URL is available (local project, no remote):**

```markdown
> **Relevant files:** `<path>` · `<path>` · `<path>`
```

Rules:
- List 3-7 files — the most important ones for the section, not every file touched
- Use the blockquote format so it renders as a distinct callout in Obsidian
- Paths are relative to the project root
- Omit this block for: `_overview.md`, `if-starting-over.md`, `resources.md`
  (these don't map to specific source files)

### Code Examples

Include short code snippets to illustrate key points — patterns, decisions,
techniques, anti-patterns. Code makes the analysis concrete and verifiable.

**Format with source URL:**

````markdown
```<language>
// <path>#L<start>-L<end>
<code snippet>
```
<sub>[source](<base-url>/blob/<branch>/<path>#L<start>-L<end>)</sub>
````

**Format without source URL:**

````markdown
```<language>
// <path>#L<start>-L<end>
<code snippet>
```
<sub>source: `<path>` lines <start>-<end></sub>
````

Rules:
- **Keep snippets short** — 5-15 lines. Trim to the essential part. Use `// ...`
  to elide irrelevant lines.
- **Always include the file path and line numbers** in a comment on the first line
  of the code block AND in the `<sub>` attribution below it
- **GitHub line links** use the `#L15-L28` fragment format for ranges
- **Choose snippets that teach** — show the pattern, the decision, or the problem.
  Don't include code just to prove you read it.
- **Implementation-heavy sections** (architecture, design-patterns, key-decisions,
  testing-strategy, gaps-vulnerabilities) need **3-5 code examples each**. These
  sections make claims about how the code works — the snippets are the evidence.
- **Analytical sections** (technology-choices, dependencies, evolution,
  if-starting-over) should include **1-3 code examples** where they ground an
  observation in real source (e.g., showing a config choice, a dependency usage
  pattern, or an evolution artifact).
- **Reference sections** (learning-path, glossary, resources, _overview) don't
  need code examples.

### Mermaid Validation

After writing all sections, validate every Mermaid diagram. Broken diagrams
render as raw syntax in Obsidian — the reader sees code instead of a chart.

Common errors to check:
- **Missing node IDs** — `A --> B` requires both A and B to be defined
- **Unclosed subgraphs** — every `subgraph` needs an `end`
- **Special characters in labels** — parentheses, brackets, and quotes in node
  labels must be wrapped: `A["Label with (parens)"]`
- **Duplicate node IDs** — each ID must be unique within a diagram
- **Direction declaration** — `graph TD`, `graph LR`, etc. must be the first line

For each diagram, mentally trace the syntax: declaration → nodes → edges →
subgraphs closed → styles applied. Fix any errors in place before moving to
Phase 3.

### `resources.md` as Collection

`resources.md` is a **collection task, not a reconstruction task**. Every link
it contains was already placed inline during section writing. To write it:

1. Grep the written section files for markdown links (`[text](url)`)
2. Deduplicate and categorize into the resources.md structure
3. Fill any gaps (a dependency mentioned in `dependencies.md` prose but
   never linked inline should get its registry/docs links here)

Do not re-research URLs — the technology URL index from Phase 1 and the
inline links from Phase 2 are the source of truth.

### Phase 3: Concepts & Index

This phase uses `_concepts/_registry.yaml` — a machine-readable index mapping
concept names to the projects that reference them. This avoids scanning every
concept file as the vault grows.

**Registry format:**

```yaml
# _concepts/_registry.yaml
# Auto-maintained by codebase-educator. Do not edit manually.
middleware:
  - expressjs--express
  - fastify--fastify
dependency-injection:
  - expressjs--express
  - spring-petclinic--spring-petclinic
testing-pyramid:
  - fastify--fastify
```

Each key is a concept name (matching the filename without `.md`). Each value
is a list of project subfolder names that reference the concept.

**Bootstrap:** If `_registry.yaml` doesn't exist (older vault), build it once
by scanning `_concepts/*.md` for "## Seen In" entries. Then proceed normally.

**Steps:**

1. **Load registry** — Read `_concepts/_registry.yaml` into memory.

2. **Collect concept list** — Gather every `[[concept-name]]` wikilink used
   across all sections written in Phase 2. Deduplicate.

3. **Concept pages** — For each concept in the list:
   - **Check the registry** (not the filesystem) to see if the concept exists.
   - If the concept is **not** in the registry → create the page using
     `references/concept-template.md`, then add the concept to the registry
     with this project as its first entry.
   - If the concept **is** in the registry → append a backlink entry under
     "## Seen In" in the existing concept page, then add this project to the
     concept's registry list.

   **Backlink format:** In concept page "Seen In" entries, always link to the
   project's `_overview.md` using the alias syntax:
   `[[<project-name>/_overview|<project-name>]]`. A bare `[[<project-name>]]`
   resolves to nothing in Obsidian since the project is a folder, not a file.

   **Batch for efficiency:** Group new concepts and write them in quick
   succession rather than interleaving reads/writes. For existing concepts
   that need a backlink appended, batch the Edit calls.

4. **Cross-project connections (bidirectional)** — Use the registry to find
   connections. No file scanning needed:
   - Filter registry entries where the value list contains **both** the new
     project and one or more other projects. These are the shared concepts.
   - For each shared concept, read the concept page's "## Seen In" section
     to get the usage descriptions for comparison (these are short — one line
     per project).
   - Write the "## Cross-Project Connections" block in the **new** project's
     `_overview.md` (already drafted in Phase 2 — update it now with real data)
   - **Also update each connected existing project's `_overview.md`**: append or
     update its "## Cross-Project Connections" section to include a reciprocal
     entry linking back to the new project. If the section doesn't exist yet
     (older briefs created before this rule), create it at the end of the file
     before any trailing `---`
   - Format for each connection block:
     ```
     Concepts shared with [[<other-project>/_overview|<Display Name>]]:
     - **<Concept>** — <how this project uses it>; <how the other project uses it>.
       <What the contrast teaches.>
     ```
   - Stage the modified existing `_overview.md` files alongside the new project
     files in the commit (Phase 4 step 3)

5. **Write registry** — Save the updated `_concepts/_registry.yaml` back to disk.
   Stage it in the Phase 4 commit.

6. **Update `_index.md`** — Add or update the entry for this project in the
   Map of Content. Include: source type, date analyzed, one-line summary,
   link to `_overview`. Also update the "Concepts by Category" section with
   any new concept pages.

   When updating `_index.md`, count the actual concept pages created (check the
   registry), not the number listed in `_overview.md`'s Concepts Introduced
   section. The two lists must agree — if they don't, fix the discrepancy before
   committing.

7. **Link validation** — Verify link integrity using the registry (no file I/O):
   - Grep all `[[wikilinks]]` from the project's section files
   - For each concept-style link (not `[[project/section]]` cross-refs),
     confirm the concept exists as a key in the registry. If it doesn't,
     either the concept page was missed in step 3 (create it now) or the
     link target is wrong (fix it to match an existing registry key)
   - For `[[project/section]]` cross-refs, validate against the known section
     filenames (fixed set: `architecture`, `technology-choices`, etc.) — no
     file existence check needed
   - Links must be **lowercase-with-hyphens** — Obsidian treats
     `[[Ones-complement]]` and `[[ones-complement]]` as different pages.
     Use the Obsidian alias syntax for display names:
     `[[ones-complement|Ones-complement]]`
   - If a broader concept already exists in the registry that covers the
     linked topic, use an alias link rather than creating a near-duplicate:
     `[[cooperative-multitasking|multitasking]]` not `[[multitasking]]`
   - **Speculative mentions:** If a section mentions a concept speculatively
     ("this may use X", "we can't confirm whether Y exists"), do NOT wikilink
     it. Only wikilink concepts that the analysis actually discusses or
     demonstrates. A wikilink is a commitment to create a concept page — don't
     link what you won't create.
   - **Every wikilink must resolve.** Zero orphan links in the final output.

### Phase 4: Commit & Push

The educator-briefs vault is a git repo. After writing all files, commit and push:

1. `cd ~/.claude/educator-briefs` (use the symlink path, not the /mnt/ path)
2. Create a branch: `git checkout -b brief/<project-name>`
   - **Multi-source:** use `brief/batch-YYYY-MM-DD` (e.g., `brief/batch-2026-03-27`)
3. Stage all new/modified files: `git add <project-name>/ _concepts/ _index.md` — this includes `_concepts/_registry.yaml` and any existing project `_overview.md` files updated with reciprocal cross-project connections
   - **Multi-source:** stage all project folders: `git add <project-1>/ <project-2>/ ... _concepts/ _index.md`
4. Commit: `git commit -m "feat(<project-name>): add educational brief"`
   - **Multi-source:** `git commit -m "feat(batch): add briefs for <project-1>, <project-2>, ..."`
5. Push: `git push -u origin brief/<project-name>`
6. Create PR: `gh pr create --title "feat(<project-name>): add educational brief" --body "New analysis of <project-name>" --fill`
7. Merge: `gh pr merge --squash --delete-branch`
8. Return to main: `git checkout main && git pull`

If the brief is an **update** to an existing project, use the commit message
`chore(<project-name>): update educational brief` instead.

### Phase 5: Report

Present to the user:
- Vault location
- **Per source:** brief summary (3-5 key findings each)
- Suggestion to open `~/.claude/educator-briefs/` as an Obsidian vault
- Total count of new concept pages created
- Any cross-project connections discovered (including between batch sources
  and with previously analyzed projects)

### Website Source — Repo Discovery & Fallback

When analyzing a website (non-GitHub HTTPS URL), try to find the source code
before falling back to external-only observation.

#### Step 1: Repo Discovery

Use WebFetch on the main URL and look for a source repository link:

1. **Page scan** — check the page content for GitHub/GitLab/Bitbucket links in:
   - Footer (common location for "View source", "GitHub" links)
   - Header/nav (open-source projects often link the repo prominently)
   - Meta tags (`<meta>` with repo URLs, `<link rel="source">`)
2. **Common URL patterns** — try `https://github.com/<domain-name-parts>` for
   obvious cases (e.g., `https://fastify.dev` → check `https://github.com/fastify/fastify`)

**Budget: one WebFetch call on the main URL.** Don't crawl subpages looking for
a repo link. If it's not on the main page or obvious from the domain, move on.

#### Step 2a: Repo Found → Pivot to Code Analysis

If a repo URL is discovered:

1. Clone it as a GitHub repo source: `git clone --depth 1 <repo-url> /tmp/educator-<name>`
2. Run the **full** analysis process (all phases, all sections)
3. Set the source URL to the discovered repo for file links and code examples
4. In `_overview.md`, note the discovery:
   ```
   **Source:** [owner/repo](https://github.com/owner/repo)
   *(discovered via [domain.com](https://domain.com))*
   ```
5. **Caveat in overview:** Note that the deployed site may differ from the
   public source — especially for monorepos or heavily customized deployments.
   If the repo is a monorepo, identify which subdirectory corresponds to the
   website and scope the analysis to that subtree.

The project subfolder is named after the **repo** (e.g., `fastify--fastify`),
not the website domain, since the analysis is based on actual source code.

#### Step 2b: No Repo Found → External Observation

If no repo is discovered, fall back to external-only analysis:

1. Use WebFetch on the main URL and 2-3 subpages
2. Analyze: frontend framework detection (view source, meta tags, script URLs),
   API patterns (network requests visible in markup), performance approach
   (lazy loading, CDN usage, SSR vs CSR indicators), security headers
3. Skip sections that require source code access: `evolution.md`, `testing-strategy.md`
4. Mark analysis as "external observation only" in the overview
5. Focus on: technology choices, observable architecture, design patterns in
   the UI/UX layer, and what the public-facing choices reveal about priorities

When writing sections for any external-observation source (website with no
repo, or any source where code access fails):

- **Code examples:** Omit. Add a note in `_overview.md` body:
  "Code examples are not included — no source code is publicly available."
- **Relevant files header:** Omit the blockquote entirely. The absence is
  self-explanatory for external observation.
- **Minimum depth:** Word count minimums still apply. The absence of code
  examples should be compensated with deeper analytical prose, Mermaid
  diagrams, and external resource links.

## Cleanup

- Remove any `/tmp/educator-*` directories created during the process
- Run `rm -rf /tmp/educator-<name>` and **check the exit code**
- If deletion fails for any reason (permissions, busy file handle, etc.):
  1. **Do not silently continue.** Tell the user the cleanup failed.
  2. Provide the exact command to run manually:
     ```
     rm -rf /tmp/educator-<name>
     ```
  3. Explain why it may have failed if the error message gives a clue
- Never leave cloned repos in `/tmp/` without the user knowing about it

## Guidelines

- **Teach, don't lecture.** Write as a knowledgeable colleague explaining their
  reasoning, not a textbook. Use analogies for complex concepts.
- **Honest assessment.** If the code is messy, say so — but explain what forces
  likely led there. Empathy for past developers is itself a lesson.
- **No moralizing.** "This uses var instead of const" is not a gap. Focus on
  architectural and design-level observations, not style lint.
- **Transferable knowledge first.** Every observation should teach a principle
  the reader can apply elsewhere. Project-specific details support the principle,
  not the other way around.
- **Difficulty calibration.** Tag sections honestly. If understanding the
  architecture requires knowing distributed systems theory, that's advanced —
  don't pretend it's beginner-friendly.
- **Living vault.** The vault grows with each analysis. Concept pages accumulate
  cross-project evidence. The value compounds over time.
- **Filename is `_overview.md`, not `overview.md`.** The underscore prefix is
  intentional — it sorts the file to the top of the project folder in Obsidian.
  **NEVER** create a file named `overview.md`. Every reference, wikilink, and
  file write must use `_overview.md`.
