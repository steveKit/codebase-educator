# gaps-vulnerabilities.md

**Goal:** Reader learns what "good enough" looks like and where the risks hide.

## Structure

1. **Architectural Gaps** — Missing components or patterns a mature system would
   typically have (no caching, no circuit breaker, no rate limiting)
2. **Scaling Concerns** — What breaks first under 10x load? 100x?
3. **Security Surface** — Observable security concerns (not a full audit —
   reference `[[security-audit]]` for that)
4. **Reliability Gaps** — Missing error handling, no retry logic, no graceful
   degradation, no health checks
5. **Operational Gaps** — Missing observability, no structured logging, no
   deployment strategy visible
6. **Specialist Audit Referrals** — Flag areas where a dedicated audit tool
   would provide deeper insight:

   | Observation | Referral | Architectural Lesson |
   |---|---|---|
   | No ARIA/semantic structure | `/a11y-audit` — `[[accessibility]]` | Accessibility is a design constraint, not a retrofit |
   | No meta tags/structured data | `/seo-audit` — `[[seo-architecture]]` | SEO is architectural when discoverability drives value |
   | Unclear dependency licenses | `/legal-audit` — `[[license-compliance]]` | Dependency choices carry legal obligations |
   | No input validation/secrets in code | `/security-audit` — `[[defense-in-depth]]` | Security must be designed in, not bolted on |

   Only include referrals where there's genuine evidence of a gap.

7. **Severity Assessment** — Which gaps matter now vs. future problems
8. **Why This Matters** — No system is complete. The skill is knowing which gaps
   to close now, accept, or monitor. Explain "appropriate engineering."

## Concepts to Link

`[[defense-in-depth]]`, `[[circuit-breaker]]`, `[[graceful-degradation]]`,
`[[observability]]`, `[[blast-radius]]`, `[[accessibility]]`,
`[[seo-architecture]]`, `[[license-compliance]]`, etc.
