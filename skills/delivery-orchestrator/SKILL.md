---
name: delivery-orchestrator
description: Orchestrate the end-to-end SDLC from idea to release and feedback. Use when starting a project, auditing a delivery flow, or choosing which skill to run next.
---

# SDLC Delivery Orchestrator

## Purpose

Coordinate the complete SDLC without turning it into bureaucracy.

## Workflow

1. Classify the work as greenfield, feature, refactor, incident, audit, or production improvement.
2. Select Lean, Standard, or Enterprise-ready depth based on team size, regulation, risk, and maturity.
3. Route to the smallest set of specialized skills needed for the current decision.
4. Keep a traceability chain from idea to requirements, BDD examples, architecture, tickets, tests, release, and production metrics.
5. When the user wants automation, select the smallest relevant harness from `harnesses/`: app idea to production, feature to production, or production feedback loop.
6. Use `agents/` role files to assign bounded specialist work with permissions, gates, and handoffs.
7. Use the scaffold script when the target repo needs the standard docs tree.
8. For spec-driven work, run the chain in order and skip steps only with a stated reason: `project-constitution` once per repo, then `design-brainstorming`, `spec-clarification`, `requirements-checklist`, `implementation-planning`, `task-breakdown`, `cross-artifact-analysis`.
9. For execution, require `branch-worktree-lifecycle` for isolation, `test-driven-development` for behavior changes, `systematic-debugging` for failures, `subagent-driven-execution` for multi-task plans, and `verification-before-completion` before any done claim.
10. Make traceability executable with `traceability-ledger` and `python scripts/sdlc_trace.py`, and close tasks only against `definition-of-done`. Pick step depth from `docs/references/depth-matrix.md`; for the full chain use `harnesses/spec-driven-delivery.yaml`.
11. For legacy replacement or merging two systems, switch to `migration-batch-orchestration` and `system-merge`, run `harnesses/legacy-migration.yaml` or `harnesses/system-merge.yaml`, and measure progress only against the frozen registry.
12. After a build pass, run `spec-convergence` to append remaining work, and `receiving-code-review` when feedback arrives.

## Outputs

- Primary artifacts: `docs/governance/decision-log.md, docs/governance/risk-register.md, docs/delivery/roadmap.md, docs/delivery/production-readiness.md`.
- Optional automation route through `agents/`, `harnesses/`, `pipelines/`, and `integrations/` when the user wants autonomous or semi-autonomous execution.

## Guardrails

- Do not start coding until the expected behavior, affected architecture/data boundaries, tests, and validation path are explicit.
- Do not delegate production deployment, secrets, destructive migrations, customer data, payments, or customer-facing communication without explicit human approval.
