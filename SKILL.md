---
name: codebase-educator
description: Analyze a codebase and produce an educational brief covering architecture, technology choices, design patterns, and gaps. Use when the user asks to "educate me on this codebase", "analyze this project for learning", "explain the architecture", "what can I learn from this code", or "codebase educator".
argument-hint: "[source] — local path, GitHub URL, website URL, npm:package, pypi:package, or omit for current project"
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

Determine the source type from `$ARGUMENTS`:

| Input | Type | Method |
|---|---|---|
| No argument | Local project | Analyze current working directory |
| `/path`, `./path`, `~/path` | Local path | Analyze directory at that path |
| `https://github.com/...` | GitHub repo | Clone to `/tmp/educator-<name>`, analyze, clean up |
| `https://...` (non-GitHub) | Website | Use WebFetch to analyze observable tech and architecture |
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
- `~/.claude/educator-briefs/_index.md` — Map of Content (MOC)
- `~/.claude/educator-briefs/_concepts/` — shared concept directory

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
4. **Sample depth** — read 3-5 files from different areas of the codebase to
   understand coding style, error handling, and patterns in practice
5. **Check history clues** (if git repo) — `git log --oneline -20` for recent
   activity patterns; `git log --diff-filter=A --name-only --format="" | head -30`
   for file creation order (reveals evolution)

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

As you write each section, link to external resources where they add value:

- **`technology-choices.md`** — Link to official docs/homepage for each major
  technology choice (language, framework, database, etc.)
- **`design-patterns.md`** — Link to authoritative pattern references (e.g.,
  refactoring.guru, language-specific pattern guides) for patterns discussed
- **`dependencies.md`** — Link to each notable dependency's homepage, docs, and
  repo (npm/PyPI/crates.io page, GitHub repo, official docs site)
- **`testing-strategy.md`** — Link to testing framework docs
- **`gaps-vulnerabilities.md`** — Link to relevant guides for addressing gaps
  (e.g., OWASP guides, caching strategy articles)
- **`learning-path.md`** — Link to prerequisite reading for advanced topics

Format inline links as standard markdown: `[Express docs](https://expressjs.com/en/api.html)`.
Place them naturally in the text where a reader would want to dive deeper, not
dumped in a list at the end of a paragraph.

#### URL Construction

For well-known registries, construct URLs directly — these follow predictable patterns:

| Registry | Pattern | Example |
|---|---|---|
| npm | `https://www.npmjs.com/package/<name>` | `https://www.npmjs.com/package/express` |
| PyPI | `https://pypi.org/project/<name>/` | `https://pypi.org/project/flask/` |
| crates.io | `https://crates.io/crates/<name>` | `https://crates.io/crates/serde` |
| pkg.go.dev | `https://pkg.go.dev/<module>` | `https://pkg.go.dev/net/http` |
| GitHub | `https://github.com/<owner>/<repo>` | `https://github.com/expressjs/express` |

For official documentation sites, use WebFetch to verify the URL resolves before
including it. If a docs URL can't be verified, link to the registry page instead —
a working link to the right package is better than a broken link to the docs.

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
- **Every section should have at least 1-2 code examples** where the section
  discusses concrete implementation details (architecture, design-patterns,
  key-decisions, testing-strategy, gaps-vulnerabilities, evolution)
- Sections that are more analytical than code-specific (technology-choices,
  dependencies, if-starting-over, learning-path, glossary) may include examples
  where they help but aren't required to

### Phase 3: Concepts & Index

1. **Concept pages** — For every `[[concept-name]]` wikilink used in the
   analysis, check if `~/.claude/educator-briefs/_concepts/<concept-name>.md`
   exists. If not, create it using the template in `references/concept-template.md`.
   If it already exists, **append** a backlink entry under "## Seen In" listing
   this project and how it uses the concept.

2. **Cross-project connections (bidirectional)** — After writing all concept
   backlinks, identify which existing projects share concepts with the new one:
   - Scan `_concepts/` for concept pages whose "## Seen In" section now lists
     **both** the new project and one or more existing projects
   - For each connection found, build a comparison entry: concept name, how each
     project uses it, and what the contrast teaches
   - Write the "## Cross-Project Connections" block in the **new** project's
     `_overview.md` (this already happens in Phase 2 via section-guide.md)
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

3. **Update `_index.md`** — Add or update the entry for this project in the
   Map of Content. Include: source type, date analyzed, one-line summary,
   link to `_overview`.

4. **Link validation** — After writing all files, verify link integrity:
   - Grep all `[[wikilinks]]` from the project's section files
   - For each concept-style link (not `[[project/section]]` cross-refs),
     confirm a matching file exists in `_concepts/`. If it doesn't, either
     the concept page was missed in step 1 (create it now) or the link
     target is wrong (fix it to match the existing concept file)
   - Links must be **lowercase-with-hyphens** — Obsidian treats
     `[[Ones-complement]]` and `[[ones-complement]]` as different pages.
     Use the Obsidian alias syntax for display names:
     `[[ones-complement|Ones-complement]]`
   - If a broader concept file exists that covers the linked topic, use an
     alias link rather than creating a near-duplicate:
     `[[cooperative-multitasking|multitasking]]` not `[[multitasking]]`
   - **Every wikilink must resolve.** Zero orphan links in the final output.

### Phase 4: Commit & Push

The educator-briefs vault is a git repo. After writing all files, commit and push:

1. `cd ~/.claude/educator-briefs` (use the symlink path, not the /mnt/ path)
2. Create a branch: `git checkout -b brief/<project-name>`
3. Stage all new/modified files: `git add <project-name>/ _concepts/ _index.md` — also stage any existing project `_overview.md` files that were updated with reciprocal cross-project connections
4. Commit: `git commit -m "feat(<project-name>): add educational brief"`
5. Push: `git push -u origin brief/<project-name>`
6. Create PR: `gh pr create --title "feat(<project-name>): add educational brief" --body "New analysis of <project-name>" --fill`
7. Merge: `gh pr merge --squash --delete-branch`
8. Return to main: `git checkout main && git pull`

If the brief is an **update** to an existing project, use the commit message
`chore(<project-name>): update educational brief` instead.

### Phase 5: Report

Present to the user:
- Vault location
- Brief summary (3-5 key findings)
- Suggestion to open `~/.claude/educator-briefs/` as an Obsidian vault
- Count of new concept pages created
- Any cross-project connections discovered (concepts shared with previously analyzed projects)

### Website Source — Modified Process

When analyzing a website (non-GitHub HTTPS URL):

1. Use WebFetch on the main URL and 2-3 subpages
2. Analyze: frontend framework detection (view source, meta tags, script URLs),
   API patterns (network requests visible in markup), performance approach
   (lazy loading, CDN usage, SSR vs CSR indicators), security headers
3. Skip sections that require source code access: `evolution.md`, `testing-strategy.md`
4. Mark analysis as "external observation only" in the overview
5. Focus on: technology choices, observable architecture, design patterns in
   the UI/UX layer, and what the public-facing choices reveal about priorities

## Cleanup

- Remove any `/tmp/educator-*` directories created during the process
- Never leave cloned repos in `/tmp/`

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
