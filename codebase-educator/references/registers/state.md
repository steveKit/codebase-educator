# State Register

**File:** `/tmp/educator-<name>/state.yaml`
**Written by:** Orchestrator (main session)
**Read by:** All phases, resume logic

The state register tracks workflow progress and enables restartability.
Write after every state transition. On resume, read this file to determine
where to pick up.

## Schema

```yaml
# /tmp/educator-<name>/state.yaml
source: expressjs--express          # project subfolder name
source-type: github                 # github | local | npm | pypi | website
source-url: https://github.com/expressjs/express
clone-path: /tmp/educator-expressjs--express  # null for local projects
state: WRITING                      # current state (see state machine below)
started: 2026-03-30T14:00:00Z
last-transition: 2026-03-30T14:12:00Z

# Phase 1 outputs (populated after GATHERED)
gather-register: /tmp/educator-expressjs--express/gather.yaml
url-index: /tmp/educator-expressjs--express/url-index.yaml

# Phase 1.5 output (populated after ASSESSED)
quality-register: /tmp/educator-expressjs--express/quality.yaml

# Phase 2 progress (updated per section)
sections:
  architecture: done          # pending | in-progress | done
  technology-choices: done
  design-patterns: in-progress
  key-decisions: pending
  dependencies: pending
  evolution: pending
  testing-strategy: pending
  gaps-vulnerabilities: pending
  if-starting-over: pending
  learning-path: pending
  glossary: pending
  resources: pending
  overview: pending

# Phase 2.5 (populated after SWEPT)
concept-sweep-done: false

# Phase 3 (populated after CONCEPTS_DONE)
concepts-created: []          # list of new concept page names
concepts-updated: []          # list of existing concepts with new backlinks
connections-added: []         # list of cross-project connection pairs

# Phase 4 (populated after COMMITTED)
branch: brief/expressjs--express
commit-sha: null
pr-url: null
```

## State Machine

```
INIT → GATHERING → GATHERED → ASSESSED → WRITING → SECTIONS_DONE
→ SWEPT → CONCEPTS_DONE → COMMITTED → COMPLETE
```

| State | Entry condition | Exit condition |
|---|---|---|
| INIT | Skill invoked | Source resolved |
| GATHERING | Source type detected | All Phase 1 data collected, registers written |
| GATHERED | gather + url-index registers on disk | Quality assessment complete |
| ASSESSED | quality register on disk | First section write begins |
| WRITING | At least one section in-progress | All 13 sections done |
| SECTIONS_DONE | All sections done | Concept sweep complete |
| SWEPT | concept-sweep-done: true | Phase 3 concepts + index done |
| CONCEPTS_DONE | concepts/registry/index updated | Git commit + push + PR done |
| COMMITTED | branch/sha/pr populated | Report presented to user |
| COMPLETE | Report shown | Terminal state |

## Resume Logic

On skill invocation, check for `/tmp/educator-<name>/state.yaml`:
- If it exists and state is not COMPLETE, offer to resume
- Read the state, skip completed phases, continue from current state
- If registers are missing for the current state, fall back to the
  previous state that produces them
