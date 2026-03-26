# Concept Page Template

Use this template when creating new concept pages in `~/.claude/educator-briefs/_concepts/`.

Filename: `<concept-name>.md` (lowercase, hyphenated).

---

## Template

```markdown
---
concept: <Concept Name>
category: <architecture | pattern | philosophy | practice | principle>
difficulty: <beginner | intermediate | advanced>
---

# <Concept Name>

<One-paragraph plain-language explanation. No jargon. Use an analogy if it
helps. The reader encountering this for the first time should walk away
understanding the core idea.>

## When You'd Use It

<2-3 sentences on the problem this concept solves. What goes wrong without it?>

## How It Works

<Brief explanation of the mechanics. Keep it general — this isn't tied to any
specific language or framework. Include a small Mermaid diagram if it helps
(see `references/diagram-guide.md` for syntax patterns).>

## Trade-Offs

| Benefit | Cost |
|---------|------|
| <what you gain> | <what you pay> |

## Common Mistakes

- <Mistake 1 — how the concept is often misapplied>
- <Mistake 2>

## Seen In

- [[<project-name>]] — <one-line description of how this project uses/demonstrates the concept>

## Further Reading

- <Title> by <Author> — <one-line description of why this resource is good>
- <URL or book reference>
```

---

## Guidelines

- **Keep it evergreen.** Concept pages are not tied to any specific project.
  They should make sense on their own.
- **The "Seen In" section grows.** Every time an analysis references this
  concept, add a line. This is how cross-project connections form.
- **Further Reading is curated.** 2-4 resources maximum. Prefer:
  1. The original paper or article that defined the concept
  2. A practical tutorial or guide
  3. A critical/nuanced take on when NOT to use it
- **Difficulty is relative to a junior-to-mid developer.** Beginner = anyone
  can understand. Intermediate = requires some professional experience.
  Advanced = requires deep domain knowledge.
- **Categories:**
  - `architecture` — structural decisions about system shape
  - `pattern` — reusable solution to a recurring problem
  - `philosophy` — guiding belief about how software should be built
  - `practice` — concrete technique or workflow
  - `principle` — fundamental rule or law (e.g., DRY, SOLID, CAP theorem)
