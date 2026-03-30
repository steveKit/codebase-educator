# URL Index Register

**File:** `/tmp/educator-<name>/url-index.yaml`
**Written by:** Phase 1 step 7
**Read by:** All section writers, Phase 3 (resources.md collection)

Single source of truth for all technology links. Section writers pull
from this — no ad hoc URL construction during writing.

## Schema

```yaml
# /tmp/educator-<name>/url-index.yaml
# One entry per technology/library/tool in the stack.

express:
  homepage: https://expressjs.com
  docs: https://expressjs.com/en/api.html
  registry: https://www.npmjs.com/package/express
  repo: https://github.com/expressjs/express
  category: framework         # language | framework | database | library | tool | testing

mocha:
  homepage: https://mochajs.org
  docs: https://mochajs.org/#getting-started
  registry: https://www.npmjs.com/package/mocha
  repo: https://github.com/mochajs/mocha
  category: testing

node:
  homepage: https://nodejs.org
  docs: https://nodejs.org/docs/latest/api/
  repo: https://github.com/nodejs/node
  category: language
```

## URL Construction Rules

For well-known registries, construct directly:

| Registry | Pattern |
|---|---|
| npm | `https://www.npmjs.com/package/<name>` |
| PyPI | `https://pypi.org/project/<name>/` |
| crates.io | `https://crates.io/crates/<name>` |
| pkg.go.dev | `https://pkg.go.dev/<module>` |
| GitHub | `https://github.com/<owner>/<repo>` |

For official docs of well-known technologies, construct directly.
Only use WebFetch when the docs URL isn't obvious from the name or
registry metadata. When uncertain, link to the registry page.

## Coverage

**Minimum:** language runtime, framework, database/datastore, major
libraries (imported in 3+ files), build tools, test framework.
Skip trivial single-use utilities.
