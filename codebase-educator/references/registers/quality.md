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

# Per-section quality notes (pre-computed for frontmatter)
section-notes:
  architecture: "Strong layered design worth studying"
  technology-choices: "Mature, well-reasoned choices"
  design-patterns: "Clean middleware pattern; some forced abstractions in router"
  key-decisions: "Key decisions well-evidenced in code structure"
  gaps-vulnerabilities: "Security surface needs observation markers"
  dependencies: "Minimal, well-maintained dependency set"
  evolution: "10+ years of clean evolution visible"
  testing-strategy: "Coverage gaps in integration tests — mark as cautionary"
  if-starting-over: "Strong foundation; few changes needed"
  learning-path: "Clear module boundaries make this easy to navigate"
  glossary: "Standard"
  resources: "Standard"
  overview: "Solid source — mixed only in testing and security"
```

## How Section Writers Use This

1. Read `overall-rating` to set the general tone
2. Read `section-notes.<section>` and paste into YAML frontmatter `quality-note`
3. If rating is `mixed` or `cautionary`, apply observation markers to
   every pattern claim (worth emulating / acceptable trade-off /
   cautionary example / anti-pattern)
4. If rating is `exemplary` or `solid`, markers are optional — use only
   when a specific observation deviates from the overall quality
