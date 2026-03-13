# Shipwright ⚓

### One skill to design it, build it, test it, polish it, and ship it.

Shipwright is an AI coding agent skill that covers the **complete software lifecycle** — from a rough idea to a production-ready system built to the engineering standards of Airbnb, Stripe, and Meta. 

Drop it into [Claude Code](https://github.com/anthropics/claude-code) or [Cursor](https://cursor.com) and your agent gains the ability to design architecture, implement with TDD, create distinctive frontends, run exhaustive QA across every layer, harden security, optimize performance, and ship with confidence.

---

## Why Shipwright?

Most AI coding agents are great at writing code but terrible at everything else — they skip design, ignore security, produce generic UIs, forget about accessibility, and have no concept of production readiness. You end up babysitting every decision.

Shipwright fixes this by giving your agent a structured methodology that covers everything from "I have an idea" to "it's live and bulletproof." It's the accumulated knowledge of what world-class software actually requires, distilled into a format that AI agents can follow.

---

## What It Does

Shipwright has two modes:

### 🔨 BUILD MODE — For new projects

A design-first workflow that prevents the agent from jumping to code before thinking:

**Think** (failure modes, constraints, requirements) → **Architect** (domain model, service boundaries, data patterns, API strategy) → **Plan** (task breakdown with TDD, YAGNI, small commits) → **Implement** (build with craft, no AI slop) → automatically flows into Ship Mode

### 🚢 SHIP MODE — For taking software live

A 12-phase production-readiness audit that covers every layer:

| Phase | What the Agent Audits & Fixes |
|:---|:---|
| **1. Code Health** | Dead code, dependencies, secrets, type safety, build warnings |
| **2. Backend & Scalability** | API design, database optimization, caching, connection pooling, background jobs, data integrity, circuit breakers, idempotency |
| **3. Testing** | Unit tests, integration tests, E2E tests, edge cases, stress testing, contract testing, visual regression |
| **4. Security** | Authentication, authorization, IDOR, SQL injection, XSS, CSRF, SSRF, secrets management, supply chain, headers, audit logging |
| **5. Frontend Design** | Light mode polish, dark mode polish, typography, color systems, animations, component states, design system, icons & illustrations |
| **6. Mobile** | 9 breakpoints (320px→1920px), touch targets, navigation collapse, responsive images, viewport handling |
| **7. Accessibility** | Semantic HTML, keyboard navigation, screen readers, ARIA, color contrast (WCAG 2.1 AA) |
| **8. Performance** | Bundle size, Core Web Vitals (LCP/INP/CLS), runtime optimization, server latency targets, rendering strategy |
| **9. CI/CD** | Pipeline setup, canary deployment, staging/production environments, Docker, post-deploy smoke tests |
| **10. Observability** | Structured logging, error monitoring, distributed tracing, SLOs, error budgets, health endpoints, incident runbooks |
| **11. World-Class Standards** | i18n readiness, SEO & GEO (Generative Engine Optimization), analytics, feature flags, error boundaries, chaos engineering, legal compliance |
| **12. Documentation** | README, architecture decision records, API docs, runbooks, architecture diagrams, tech debt log |

The agent doesn't just list problems — it **fixes them immediately and tests the fix** before moving on.

---

## Quick Start

### Claude Code (global — works across all your projects)

```bash
git clone https://github.com/saadjangda/shipwright.git ~/.claude/skills/shipwright
```

### Claude Code (single project)

```bash
git clone https://github.com/saadjangda/shipwright.git .claude/skills/shipwright
```

### Cursor

```bash
git clone https://github.com/saadjangda/shipwright.git .cursor/skills/shipwright
```

Then just talk to your agent:

- **"Let's build a dashboard for tracking sales"** → BUILD mode activates
- **"Make this production ready"** → SHIP mode runs the full 12-phase audit
- **"Harden the security"** → Jumps directly to Phase 4
- **"Polish the dark mode"** → Jumps directly to Phase 5
- **"Is this ready to ship?"** → Full audit

The skill activates automatically based on what you ask. No commands to memorize.

---

## How It Works Under the Hood

The skill uses a layered loading system to stay efficient:

```
shipwright/
├── SKILL.md                    # Always loaded (288 lines) — the full workflow
└── references/                 # Loaded on-demand — deep-dive checklists
    ├── architecture.md         # Domain modeling, service boundaries, data patterns
    ├── frontend-design.md      # Typography, color, motion, light/dark mode
    ├── security-hardening.md   # Security deep-dive with test payloads
    ├── mobile-responsive.md    # Cross-device QA checklist
    ├── seo-geo.md              # Technical SEO, content SEO, GEO
    └── production-readiness.md # Granular sub-steps for every phase
```

When you ask your agent to "make this production ready," it loads the 288-line `SKILL.md` which has the complete workflow. When it reaches a phase that needs deep detail — say, security hardening — it loads just that reference file. This keeps the agent's context window lean instead of dumping 1,600+ lines upfront.

---

## What Makes It Different

**It's opinionated.** Shipwright doesn't give your agent options — it tells it exactly what "good" looks like. bcrypt cost ≥12. LCP < 2.5s. Touch targets ≥44px. No `dangerouslySetInnerHTML` without DOMPurify. These aren't suggestions; they're standards.

**It's anti-AI-slop.** The frontend design section explicitly bans generic AI aesthetics — no Inter/Roboto for headings, no purple gradients on white, no cookie-cutter layouts. It pushes the agent toward distinctive, intentional design choices.

**It covers what others don't.** Most coding skills handle implementation. Shipwright also covers domain modeling (bounded contexts, aggregates, CQRS), scalability (stateless apps, read replicas, caching strategies, background jobs), reliability engineering (SLOs, error budgets, circuit breakers), GEO (Generative Engine Optimization for AI discoverability), and chaos engineering (offline testing, concurrent sessions, timezone edge cases).

**Everything is agent-actionable.** Unlike architecture docs written for humans, every item in Shipwright is something an AI coding agent can actually execute. No org design, no team topologies, no facilitation exercises — just code-level actions.

---

## Credits

Shipwright synthesizes ideas and practices from:

- [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent — structured agent development workflow (brainstorm → plan → implement)
- [Frontend Design](https://github.com/anthropics/claude-code/tree/main/plugins/frontend-design) by Anthropic — anti-AI-slop frontend aesthetics
- A comprehensive architecture reference covering DDD, CQRS, event sourcing, polyglot persistence, and reliability engineering
- Industry standards: OWASP (security), WCAG 2.1 (accessibility), Google SRE (reliability), Core Web Vitals (performance)

---

## Contributing

Found something missing? Have a better way to handle a specific check? Contributions are welcome.

1. Fork the repo
2. Make your changes
3. Submit a PR with a clear description of what you changed and why

The bar for inclusion: **is this something an AI coding agent can actually execute, and does it make shipped software meaningfully better?**

---

## License

[MIT](LICENSE) — use it however you want.
