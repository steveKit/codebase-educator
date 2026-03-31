# Quality Register

**File:** `/tmp/educator-<name>/quality.yaml`
**Written by:** Phase 1.5 (Quality Assessment)
**Read by:** All section writers

Captures the quality assessment so section writers can calibrate tone
without re-evaluating. See `references/quality-assessment.md` for the
full rubric — this register stores the results.

## Schema

```yaml
# /tmp/educator-<name>/quality.yaml
overall-rating: solid           # exemplary | solid | mixed | cautionary

signals:
  engineering-maturity: strong   # strong | mixed | weak
  maintenance-health: strong
  testing-discipline: mixed
  documentation-intent: strong
  security-awareness: mixed
  dependency-hygiene: strong

tone-guidance: |
  Frame as "learn from this, with caveats." Mostly good engineering
  with specific areas that could improve. Testing and security sections
  need observation markers.

# Per-section quality notes AND depth tiers (pre-computed)
# depth: deep | standard | light | skip
# skip-reason: only present when depth is "skip"
section-notes:
  architecture:
    note: "Strong layered design worth studying"
    depth: deep
  technology-choices:
    note: "Mature, well-reasoned choices"
    depth: standard
  design-patterns:
    note: "Clean middleware pattern; some forced abstractions in router"
    depth: deep
  key-decisions:
    note: "Key decisions well-evidenced in code structure"
    depth: standard
  gaps-vulnerabilities:
    note: "Security surface needs observation markers"
    depth: standard
  dependencies:
    note: "Minimal, well-maintained dependency set"
    depth: light
  evolution:
    note: "10+ years of clean evolution visible"
    depth: deep
  testing-strategy:
    note: "Coverage gaps in integration tests — mark as cautionary"
    depth: standard
  if-starting-over:
    note: "Strong foundation; few changes needed"
    depth: standard
  learning-path:
    note: "Clear module boundaries make this easy to navigate"
    depth: standard
  glossary:
    note: "Standard"
    depth: light
  resources:
    note: "Standard"
    depth: standard
  overview:
    note: "Solid source — mixed only in testing and security"
    depth: standard
```

## How Section Writers Use This

1. Read `overall-rating` to set the general tone
2. Read `section-notes.<section>.note` and paste into YAML frontmatter `quality-note`
3. Read `section-notes.<section>.depth` to calibrate content volume:
   - `deep` — full treatment: thorough prose, multiple code samples, diagrams,
     comparison tables. This section has rich material to work with.
   - `standard` — solid coverage with code examples and a diagram where useful.
   - `light` — essentials only. Don't pad. Better short and honest than long and thin.
   - `skip` — write a stub file with frontmatter and one-line explanation only.
4. If rating is `mixed` or `cautionary`, apply observation markers to
   every pattern claim (worth emulating / acceptable trade-off /
   cautionary example / anti-pattern)
5. If rating is `exemplary` or `solid`, markers are optional — use only
   when a specific observation deviates from the overall quality
