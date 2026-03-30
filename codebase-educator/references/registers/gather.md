# Gather Register

**File:** `/tmp/educator-<name>/gather.yaml`
**Written by:** Phase 1 (Gather)
**Read by:** Phase 1.5, Phase 2 section writers, Phase 2.5, Phase 3

Captures all structured data from codebase scanning. Section writers read
this instead of re-reading source files — this is the primary token savings.

## Schema

```yaml
# /tmp/educator-<name>/gather.yaml
project-name: expressjs--express
source-type: github
base-url: https://github.com/expressjs/express  # null if no remote
default-branch: main

# Structure snapshot
top-level-files:
  - package.json
  - index.js
  - README.md
  # ...full ls output

entry-points:
  - path: lib/express.js
    role: main module export
  - path: lib/application.js
    role: app factory

config-files:
  - path: package.json
    summary: "Node.js project, MIT license, 72 deps"
  - path: .eslintrc.yml
    summary: "ESLint with standard config"

# Dependency graph (structured, not raw imports)
dependencies:
  runtime:
    - name: accepts
      version: ~1.3.8
      purpose: content negotiation
      import-count: 1
    # ...
  dev:
    - name: mocha
      version: ^10.2.0
      purpose: test framework
    # ...

# Module structure
modules:
  - name: lib/
    role: core framework
    files: [express.js, application.js, request.js, response.js, router/]
  - name: test/
    role: test suite
    files: [app.test.js, req.test.js, ...]

# Architecture observations (condensed from file reads)
layers:
  - name: routing
    files: [lib/router/index.js, lib/router/route.js, lib/router/layer.js]
    notes: "Chain-of-responsibility pattern, middleware pipeline"
  - name: request/response
    files: [lib/request.js, lib/response.js]
    notes: "Extends Node http.IncomingMessage/ServerResponse"

# Key file summaries (replacing raw file content in context)
sampled-files:
  - path: lib/application.js
    lines: 644
    summary: |
      App factory. Creates express app objects. Key patterns:
      - Mixin-based: app is a plain function with methods mixed in
      - Lazy initialization of router (created on first use)
      - Settings system via app.set()/app.get()
    key-snippets:
      - lines: "38-55"
        description: "app factory function — mixin pattern"
      - lines: "180-210"
        description: "lazy router initialization"
  # ... more sampled files

# Test file observations
test-files:
  - path: test/app.test.js
    framework: mocha
    style: "BDD (describe/it), supertest for HTTP assertions"
    coverage-focus: "app configuration, routing, error handling"
  - path: test/req.cookies.js
    framework: mocha
    style: "focused unit tests"
    coverage-focus: "cookie parsing edge cases"

# History clues
git-history:
  recent-commits: |
    abc1234 fix: handle empty Content-Type header
    def5678 feat: add app.router getter
    ...
  file-creation-order:
    - lib/express.js
    - lib/application.js
    # ... oldest to newest
  age-estimate: "10+ years, very mature"

# Documentation found
docs:
  readme: "Comprehensive — installation, API overview, examples"
  architecture-docs: null
  api-docs: "External at expressjs.com"
  inline-comments: "Moderate — concentrated in complex areas"
```

## Key Principle

The gather register replaces raw file contents in the context window.
Instead of carrying 500+ lines of source code through all phases, section
writers get structured summaries with pointers to specific line ranges
they can read on demand.

**A section writer should be able to write its section from:**
1. This register (structure, summaries, key observations)
2. The url-index register (links)
3. The quality register (tone calibration)
4. Targeted reads of 2-5 specific files/line ranges (for code examples)

This reduces per-section context from "everything gathered" to
"structured summary + targeted reads."
