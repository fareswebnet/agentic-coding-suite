# Method Sources

Skills added in the speckit and superpowers enrichment adapt ideas into original wording; no text is copied.

| Skill | Inspiration | Idea adapted |
|---|---|---|
| `project-constitution` | GitHub Spec Kit `constitution` | Versioned principles, sync-impact sweep, scope guard |
| `spec-clarification` | Spec Kit `clarify` | Bounded one-at-a-time interview with recommended answers |
| `requirements-checklist` | Spec Kit `checklist` | Checklists that test requirement quality |
| `task-breakdown` | Spec Kit `tasks`, Superpowers `writing-plans` | Story-grouped, parallel-marked, exact-path tasks |
| `cross-artifact-analysis` | Spec Kit `analyze` | Read-only severity-ranked consistency audit |
| `spec-convergence` | Spec Kit `converge` | Assess code vs spec and append remaining tasks |
| `design-brainstorming` | Superpowers `brainstorming` | Hard design gate before implementation |
| `test-driven-development` | Superpowers `test-driven-development` | Verified red, minimal green, refactor |
| `systematic-debugging` | Superpowers `systematic-debugging` | Root cause first, three-strikes rule |
| `verification-before-completion` | Superpowers `verification-before-completion` | Evidence before claims |
| `subagent-driven-execution` | Superpowers `subagent-driven-development`, `dispatching-parallel-agents` | Fresh agent per task, two-stage review |
| `branch-worktree-lifecycle` | Superpowers `using-git-worktrees`, `finishing-a-development-branch` | Isolation and explicit finish options |
| `receiving-code-review` | Superpowers `receiving-code-review` | Verify feedback before implementing |

## Beyond the sources

These skills have no direct upstream equivalent and extend the idea of both projects.

| Skill | What it adds |
|---|---|
| `traceability-ledger` | Id-based chain checked by `scripts/sdlc_trace.py`, extended to release and metrics |
| `definition-of-done` | Depth- and risk-scaled criteria, each with a named evidence type and waiver rules |
| `agent-context-engineering` | Designed context packs, subagent briefs, handoff notes, and honest memory |
| `tasks-to-issues` | Approval-gated tracker sync with dependency links and reconciliation |
| `delivery-retrospective` | Evidence-based loop that changes constitution, templates, skills, and CI |
| `skill-evaluation` | Rubric-scored baseline-versus-skill evaluation with version history |

## Migration and specialized skills

The migration layer generalizes, in English and project-agnostic form, the practice of a real multi-source migration program (frozen registry as denominator, vertical batches, independent verifier, reachability ratchet, cumulative non-regression, separate visas per source). Project-specific paths, tools, and business rules were removed; commands here use `scripts/sdlc_registry.py`.

| Skill | Basis |
|---|---|
| `legacy-inventory-registry`, `migration-batch-orchestration` | Registry-driven migration programs |
| `legacy-reverse-engineering`, `characterization-testing` | Spec mining, golden master, differential testing |
| `independent-verification`, `reachability-anti-simulation` | Independent audit, mutation testing, ratchet gates |
| `system-merge`, `data-migration-reconciliation`, `cutover-rollback` | Multi-source convergence, data reconciliation, strangler and parallel run |
| `performance-budgets` | Web performance budgets, SRE objectives |
| `accessibility-engineering` | WCAG 2.2 |
| `api-versioning-deprecation` | Semantic versioning, consumer-driven contracts |

Both upstream projects are MIT-licensed; review their current licenses before copying any text.
