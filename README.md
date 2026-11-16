# Agentic Coding Suite

*Reinforcement for AI-driven delivery: evidence, verification, no unproven claims.*

Agentic Coding Suite plugs into your existing delivery process, not in place of it: an agent skill suite (58 skills, agent roles, harnesses, evals) for the full product-tech software lifecycle, with independent verification and measured effect.

This repository packages 58 skills and optional presets to help a team move from idea to production learning:

`idea -> discovery -> PRD -> domain model -> bounded contexts -> BDD -> UX/UI -> architecture/data -> contracts -> backlog -> implementation -> tests -> release -> observability -> incidents/support -> improvement`

The suite is designed for startups and scale-ups by default, but each skill can be used in Lean, Standard, or Enterprise-ready depth. It now also includes an agent-factory layer for highly automated coding workflows across Codex, Claude Code, Hermes, OpenClaw, GitHub Actions, MCP connectors, or a custom orchestrator.

## What This Is

- A repo-first SDLC operating system for AI-assisted product and engineering work.
- A set of portable agent skills using `SKILL.md` folders.
- A vendor-neutral agent orchestration layer for moving from app idea to PR, preview, release, and production learning.
- A standards-informed implementation layer over ISO/IEC/IEEE, DDD, BDD, OWASP, NIST, SLSA, DORA, OpenTelemetry, SRE, FinOps, WCAG, OpenAPI, AsyncAPI, and AI risk practices.

## What This Is Not

- Not legal, accounting, financial, compliance, investment, or security certification advice.
- Not a replacement for qualified review in regulated or high-risk contexts.
- Not a command to over-document every project. Use the depth that fits the risk.
- Not a license to run production changes, secrets, destructive migrations, payments, or customer-impacting communications without human approval.

## Install

One installer covers every runtime. Skills go where each runtime discovers them, and the only repository memory file is `AGENTS.md` (a managed block that never overwrites your own content).

```bash
python scripts/install.py --runtime codex|claude|antigravity|generic --target /path/to/repo --agents-md --with-kit
python scripts/install.py --runtime codex --scope user        # user-level skills
python scripts/install.py --runtime claude --target . --native-roles   # also .claude/agents subagents
```

| Runtime | Project skills folder | Invocation |
|---|---|---|
| Codex | `.agents/skills` | `$coding-<name>` |
| Claude Code | `.claude/skills` (or plugin via `.claude-plugin/`) | `/coding-<name>` (standalone install) or `/coding:<name>` (plugin), or by description |
| Antigravity | `.agents/skills` (or plugin via `plugin.json`) | mention the skill name |
| Open-source or other LLM | `.agents/skills`, plus `python scripts/skill_index.py` | name the skill in the prompt |

Small models and runtimes without skills: `compact/` holds shortened skills, `python scripts/sdlc_runner.py` runs a gated loop against any OpenAI-compatible endpoint, and `python scripts/run_evals.py` measures whether a model follows the skills (see `evals/README.md` and `integrations/generic-llm/`).

Restart the agent session so new skills are discovered. Fallbacks for missing tools (subagents, worktrees, MCP) are in `docs/references/runtime-capabilities.md`; per-runtime notes are in `integrations/`.

## Skills

| Skill | Purpose |
|---|---|
| `delivery-orchestrator` | Orchestrate the complete lifecycle and route to the right skill. |
| `idea-framing` | Clarify raw product ideas. |
| `market-user-discovery` | Research users, market, and evidence. |
| `prd-requirements` | Write precise, testable PRDs and requirements. |
| `domain-modeling` | Model business concepts, rules, and invariants. |
| `bounded-context-map` | Map DDD context boundaries and ownership. |
| `bdd-specifications` | Specify behavior with examples and Gherkin. |
| `ux-product-design` | Design user journeys, flows, and states. |
| `ui-design-system` | Plan UI components, tokens, and accessibility. |
| `architecture-rfc` | Create architecture RFCs and ADR candidates. |
| `application-architecture` | Design modules, layers, services, and boundaries. |
| `data-architecture` | Design data ownership, schemas, migrations, and audit. |
| `security-privacy-compliance` | Integrate AppSec, privacy, compliance, and threat modeling. |
| `ai-coding-governance` | Govern AI-assisted coding workflows. |
| `ai-product-architecture` | Design LLM, RAG, agent, eval, safety, and AI observability. |
| `delivery-roadmap-backlog` | Slice roadmap and backlog into executable work. |
| `implementation-planning` | Plan safe code changes. |
| `code-change-execution` | Execute scoped repository changes. |
| `test-strategy` | Design quality strategy and tests by risk. |
| `code-review` | Review code against quality gates. |
| `ci-cd-platform` | Design delivery platform and CI/CD. |
| `observability-reliability` | Design SLOs, telemetry, alerts, and runbooks. |
| `incident-support-feedback` | Close incident, support, and feedback loops. |
| `scalability-cost` | Balance performance, scale, and FinOps. |
| `product-analytics` | Design metrics, tracking, experiments, and data quality. |
| `documentation` | Keep documentation and onboarding useful. |
| `third-party-ip-risk` | Review dependencies, licenses, vendors, and IP risk. |
| `project-constitution` | Define non-negotiable engineering principles. |
| `spec-clarification` | Resolve spec ambiguity before planning. |
| `requirements-checklist` | Unit-test the requirements themselves. |
| `task-breakdown` | Create dependency-ordered executable tasks. |
| `cross-artifact-analysis` | Audit spec, plan, tasks for consistency. |
| `spec-convergence` | Reconcile built code against the spec. |
| `design-brainstorming` | Design gate before any implementation. |
| `test-driven-development` | Red, green, refactor with proof. |
| `systematic-debugging` | Root cause before any fix. |
| `verification-before-completion` | Evidence before any done claim. |
| `subagent-driven-execution` | Execute tasks with fresh agents and reviews. |
| `branch-worktree-lifecycle` | Isolate work and finish branches safely. |
| `receiving-code-review` | Evaluate review feedback before implementing. |
| `legacy-inventory-registry` | Freeze everything the legacy system exposes. |
| `legacy-reverse-engineering` | Extract business rules and behavior from legacy code. |
| `characterization-testing` | Pin legacy behavior with oracles and golden masters. |
| `migration-batch-orchestration` | Drive a migration in verified vertical batches. |
| `independent-verification` | Refute porting claims in a separate context. |
| `reachability-anti-simulation` | Block unrouted code and faked clients. |
| `system-merge` | Merge two systems or estates into one target. |
| `data-migration-reconciliation` | Move and reconcile data with proof. |
| `cutover-rollback` | Rehearse go-live, parallel run, and retreat. |
| `performance-budgets` | Set and enforce measurable performance budgets. |
| `accessibility-engineering` | Build and verify WCAG-conformant interfaces. |
| `api-versioning-deprecation` | Evolve APIs without breaking consumers. |

## Spec-Driven and Disciplined Execution

Inspired by GitHub Spec Kit and Superpowers (see `docs/references/method-sources.md`), the suite adds a spec-driven chain and execution discipline:

`constitution -> brainstorm -> clarify -> checklist -> plan -> tasks -> analyze + trace -> TDD execution (subagent-driven, worktree-isolated) -> verify against definition of done -> review -> converge -> retrospective`

- Spec chain: `project-constitution`, `design-brainstorming`, `spec-clarification`, `requirements-checklist`, `task-breakdown`, `cross-artifact-analysis`, `spec-convergence`.
- Execution discipline: `test-driven-development`, `systematic-debugging`, `verification-before-completion`, `subagent-driven-execution`, `branch-worktree-lifecycle`, `receiving-code-review`.
- Beyond both sources: `traceability-ledger` (script-checked requirement-to-test chain), `definition-of-done` (depth-scaled, evidence-first), `agent-context-engineering`, `tasks-to-issues`, `delivery-retrospective`, `skill-evaluation`.
- Meta: `skill-evaluation`.

Supporting assets: `docs/references/depth-matrix.md` (which steps at Lean, Standard, Enterprise-ready), `harnesses/spec-driven-delivery.yaml`, `pipelines/spec-driven-delivery.md`, `scripts/sdlc_trace.py` with `scripts/test_sdlc_trace.py`, and a worked fixture in `examples/spec-driven-trace/`.

```bash
python scripts/sdlc_trace.py --target examples/spec-driven-trace --strict
```

## Legacy Migration and System Merge

Migrating a legacy estate, or merging two, fails when progress is counted on what was written instead of what existed, and when the author of a change is its only judge. This layer fixes both:

`frozen registry -> batches -> reverse engineering -> oracles -> test-first build -> reachability ratchet -> independent verification and visa -> rehearsed cutover -> lessons`

- Skills: `legacy-inventory-registry`, `legacy-reverse-engineering`, `characterization-testing`, `migration-batch-orchestration`, `independent-verification`, `reachability-anti-simulation`, `system-merge`, `data-migration-reconciliation`, `cutover-rollback`.
- Agents: `legacy-analyst` and `independent-verifier` (the verifier runs in a separate context and cannot visa its own work).
- Harnesses: `harnesses/legacy-migration.yaml`, `harnesses/system-merge.yaml`; pipelines `pipelines/legacy-migration.md`, `pipelines/system-merge.md`.
- Tool: `scripts/sdlc_registry.py` (init, check, report, sample, claim, visa, ratchet), tested by `scripts/test_sdlc_registry.py`.

```bash
python scripts/sdlc_registry.py report docs/migration/registry.csv
python examples/legacy-migration-invoicing/run_example.py   # full worked example
```

Specialized quality skills: `performance-budgets`, `accessibility-engineering`, `api-versioning-deprecation`.

## Presets

- `fintech-regulated`
- `b2b-saas`
- `ai-agent-product`
- `mobile-offline`
- `marketplace`
- `data-product`
- `enterprise-ready`
- `legacy-migration`
- `system-merge`
- `healthtech-regulated`
- `public-sector`
- `ecommerce-retail`

Presets are stored in `presets/` and tune skill depth for domain-specific projects.

## Agent Factory

The automation layer separates method from runtime:

- `skills/`: reusable SDLC methods and guardrails.
- `agents/`: specialist role definitions, permissions, gates, and handoffs.
- `harnesses/`: executable state machines for idea-to-production, feature-to-production, spec-driven delivery, legacy migration, system merge, and production feedback.
- `pipelines/`: human-readable operating procedures for PRs, releases, and production learning.
- `integrations/`: adapters for Codex, Claude Code, Hermes, OpenClaw, GitHub Actions, and MCP.
- `data/agent-runtime-matrix.csv`: runtime and gate mapping by agent.

Primary agents:

| Agent | Role |
|---|---|
| `orchestrator` | Split work, select skills, enforce gates, and route handoffs. |
| `product-strategist` | Clarify idea, evidence, PRD, MVP, roadmap, and metrics. |
| `domain-architect` | Model domain rules, bounded contexts, and BDD examples. |
| `ux-ui-designer` | Define journeys, UI states, accessibility, and tracking. |
| `solution-architect` | Decide architecture, data, APIs, identity, and tradeoffs. |
| `fullstack-builder` | Implement approved vertical slices in isolated branches or worktrees. |
| `qa-test-engineer` | Build acceptance, regression, and risk-based test evidence. |
| `security-reviewer` | Review AppSec, privacy, tenants, dependencies, and AI risk. |
| `devops-sre` | Prepare CI/CD, reliability, observability, rollback, and runbooks. |
| `code-reviewer` | Make the merge-gate decision from diff, specs, tests, and risk. |
| `release-manager` | Own go/no-go, rollout, post-release checks, and production learning. |
| `legacy-analyst` | Inventory legacy sources, freeze the registry, extract rules, and classify convergence. |
| `independent-verifier` | Refute claimed work in a separate context and grant or refuse visas. |

## Automated Workflows

Idea to PR:

```text
Use harnesses/app-idea-to-production.yaml.
Start from this app idea, create the product docs, BDD, architecture, implementation plan, then implement the first vertical slice.
Stop before production deployment.
```

Feature to production:

```text
Use harnesses/feature-to-production.yaml.
Route this feature through behavior, impact, implementation, QA, security, review, and release readiness.
```

Spec-driven delivery:

```text
Use harnesses/spec-driven-delivery.yaml.
Take this feature through design, clarified spec, traceable tasks, test-first build, verification evidence, convergence, and retrospective.
```

Production feedback loop:

```text
Use harnesses/production-feedback-loop.yaml.
Turn this incident, support signal, analytics anomaly, or reliability issue into a fix, learning, or product decision.
```

Human approval remains required for product scope, durable architecture choices, dependency additions, secrets, customer data, payments, destructive migrations, and production deployment.

## Examples

- `examples/fintech-cfo-b2b/sdlc-case-study.md`: compact case study for a fintech CFO product.
- `examples/agent-factory-idea-to-pr/`: complete automation example showing how a raw app idea becomes a task graph, artifact map, simulated PR, and review gate across Codex, Claude Code, and Hermes prompts.
- `examples/hermes-releasepilot-signal-clustering/`: complete Hermes operating pack for a ReleasePilot Signal Clustering feature, including profile context, project skills, task graph, BDD, red tests, feature flag, simulated PR, review gate, production follow-up, and ROI loop.

## Scaffold Project Docs

From this repository:

```bash
python skills/delivery-orchestrator/scripts/scaffold_sdlc_docs.py --target /path/to/project
```

This copies reusable templates from `templates/docs/` into the target project's `docs/` tree without overwriting existing files unless `--force` is passed.

## Sources

See:

- `docs/references/standards-map.md`
- `docs/references/market-skill-benchmark.md`
- `docs/references/sourcing-matrix.md`
- `docs/references/agent-runtime-matrix.md`

## Runtime Adapters

- `integrations/codex/README.md`
- `integrations/claude-code/README.md`
- `integrations/hermes/README.md`
- `integrations/openclaw/README.md`
- `integrations/github-actions/README.md`
- `integrations/mcp/README.md`

## License

MIT
