---
name: codebase-educator
description: Analyze a codebase and produce an educational brief covering architecture, technology choices, design patterns, and gaps. Use when the user asks to "educate me on this codebase", "analyze this project for learning", "explain the architecture", "what can I learn from this code", or "codebase educator".
argument-hint: "[source] — local path, GitHub URL, website URL, npm:package, pypi:package, or omit for current project"
disable-model-invocation: true
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, WebFetch]
---

# Codebase Educator

Analyze a source and produce a structured educational brief as an Obsidian-compatible
vault in `~/.claude/educator-briefs/`.

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
├── overview.md              # Executive summary + wikilinks to all sections
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
└── glossary.md              # Domain and technical terms defined in context
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
- Prefer ASCII diagrams for architecture visualization — they render everywhere
- Be honest about uncertainty: "This appears to be..." rather than asserting
  intent you can't verify from code alone

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
12. `overview.md` (last — it summarizes everything above)

### Phase 3: Concepts & Index

1. **Concept pages** — For every `[[concept-name]]` wikilink used in the
   analysis, check if `~/.claude/educator-briefs/_concepts/<concept-name>.md`
   exists. If not, create it using the template in `references/concept-template.md`.
   If it already exists, **append** a backlink entry under "## Seen In" listing
   this project and how it uses the concept.

2. **Update `_index.md`** — Add or update the entry for this project in the
   Map of Content. Include: source type, date analyzed, one-line summary,
   link to overview.

### Phase 4: Report

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
