# Source Quality Assessment

Evaluate the source's engineering quality at the end of Phase 1 (Gather).
This assessment calibrates the tone and framing of every section in the brief.

---

## Quality Signals

Evaluate each signal on a three-point scale: **strong**, **mixed**, **weak**.

### 1. Engineering Maturity

- **Strong:** Consistent code style, clear module boundaries, separation of
  concerns, appropriate abstraction levels, predictable file organization
- **Mixed:** Some well-structured areas alongside disorganized ones, inconsistent
  abstraction levels, partial refactors
- **Weak:** God files, circular dependencies, no discernible architecture,
  copy-paste duplication, mixed responsibilities everywhere

### 2. Maintenance Health

- **Strong:** Recent meaningful commits, dependencies within 1-2 major versions
  of current, issues triaged, CI passing
- **Mixed:** Sporadic activity, some stale dependencies, partially maintained
- **Weak:** Abandoned (no commits in 12+ months), severely outdated dependencies,
  known vulnerabilities unpatched

### 3. Testing Discipline

- **Strong:** Tests exist for critical paths, test quality is high (testing
  behavior not implementation), CI enforces tests
- **Mixed:** Some tests exist but coverage is uneven, test quality varies,
  some brittle tests
- **Weak:** No tests, or tests that are all skipped/broken, or tests that
  only test trivial cases

### 4. Documentation Intent

- **Strong:** README explains purpose and setup, code has meaningful comments
  on non-obvious logic, API is documented
- **Mixed:** README exists but is outdated, some comments but inconsistent,
  partial documentation
- **Weak:** No README or boilerplate-only, no comments, no documentation

### 5. Security Awareness

- **Strong:** Input validation at boundaries, parameterized queries, secrets
  externalized, auth implemented correctly, security headers present
- **Mixed:** Some validation but inconsistent, some security patterns present
  but gaps exist
- **Weak:** Raw user input in queries/commands, secrets in code, no validation,
  no auth on protected resources

### 6. Dependency Hygiene

- **Strong:** Minimal dependencies, pinned versions, no known vulnerabilities,
  lock file committed
- **Mixed:** Reasonable dependency count but some unpinned or outdated,
  minor vulnerabilities
- **Weak:** Excessive dependencies, floating versions, abandoned packages,
  known critical vulnerabilities

---

## Overall Quality Rating

Aggregate the signals into one of four ratings:

### Exemplary

4+ signals strong, none weak. Frame the brief as **"learn from this"** —
this codebase demonstrates solid engineering. Highlight what makes it good
and why those choices work.

> Tone: "This project does X, and here's why it's effective..."

### Solid

3+ signals strong, at most 1 weak. Frame as **"learn from this, with caveats"** —
mostly good engineering with specific areas that could improve.

> Tone: "This project handles X well. However, in Y area, a stronger approach
> would be..."

### Mixed

A blend of strong and weak signals. Frame as **"learn selectively"** —
some areas worth emulating, others worth studying as cautionary examples.
Each section must clearly label which it is.

> Tone: "This project's approach to X is worth studying. Its approach to Y,
> however, shows what happens when..."

### Cautionary

3+ signals weak. Frame the entire brief as **"learn what not to do"** —
the educational value is in understanding the consequences of poor decisions.
Every section must present the correct alternative alongside the observation.

> Tone: "This project does X. This is problematic because [reason]. A better
> approach would be [alternative], which avoids [consequence]."

---

## Section Integration

The quality rating affects every section:

### In the overview.md

Add a prominent quality banner immediately after the one-paragraph summary:

```markdown
> **Source Quality: [Rating]**
> [One-sentence summary of the quality assessment]
> This brief frames observations accordingly — patterns marked as strengths
> have been validated; patterns marked as cautionary are explained with
> better alternatives.
```

### In each section file

Add a `quality-note` property to the frontmatter:

```yaml
quality-note: "<one-line note on how quality rating affects this section>"
```

For **mixed** and **cautionary** sources, every observation must use one of
these markers:

- **Worth emulating** — This pattern is well-applied here. Explanation of why.
- **Acceptable trade-off** — Not ideal but reasonable given likely constraints.
  Explanation of what the trade-off is.
- **Cautionary example** — This pattern is problematic. Explanation of the
  problem and the better alternative.
- **Anti-pattern** — This is actively harmful. Explanation of consequences
  and the correct approach.

For **exemplary** and **solid** sources, the markers are optional — use them
only when a specific observation deviates from the overall quality level.

### In concept pages (_concepts/)

The "Seen In" backlink entry should note the quality context:

```markdown
- [[project-name]] — uses this pattern effectively (exemplary source)
- [[other-project]] — demonstrates misapplication of this pattern (cautionary source)
```

This prevents a reader from seeing "Project X uses the Repository Pattern"
and assuming it's a good implementation when it isn't.

---

## Constraints

- **Never refuse to analyze low-quality code.** Bad code is highly educational.
  The skill is in the framing, not the gatekeeping.
- **Never be cruel.** Open source maintainers are often solo, unpaid, and
  doing their best. Critique the code, not the people. "This codebase lacks X"
  not "the developers didn't know what they were doing."
- **Acknowledge context.** A weekend hackathon project and a production banking
  system have different quality expectations. Note the likely context when
  assessing.
- **Per-section granularity.** A codebase can have excellent architecture but
  terrible testing. Rate the whole, but mark sections individually when they
  deviate from the overall rating.
