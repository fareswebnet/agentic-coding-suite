---
name: migration-strategy
description: Choose and justify a modernization strategy (uplift, transform, reimagine, rehost, replace), order batches by a risk ladder, design the coexistence period, run a pilot batch, then industrialize. Use before planning batches or when a migration has no stated approach.
---

# SDLC Migration Strategy

## Purpose

Make the approach a documented decision tied to constraints, and prove it on a pilot before scaling it.

## Workflow

1. Write the drivers: availability, volumes, regulation, team skills, budget and deadline, appetite for business change, market options.
2. Compare the five trajectories against the drivers, with a rejection reason for each one dropped: uplift (newer version of the same technology), transform (new technology, same behavior), reimagine (new processes or architecture), rehost (move without recoding), replace (market product). Default to transform; choose reimagine only when processes must change, rehost or uplift only to exit a platform or support window with the debt acknowledged.
3. Justify the target architecture by constraint, not fashion; use `architecture-rfc` and `application-architecture`. A modular monolith is a valid answer.
4. Order the batches by risk ladder: consultation and reporting, low-criticality workflows, stable interfaces, non-critical batch jobs, critical business functions, then the transactional core and sensitive data with reinforced proof. Adjust with coupling from `bounded-context-map` and usage from `legacy-usage-evidence`.
5. Design coexistence: the source of truth per entity during the transition, sync direction, frozen interface contracts, routing unit, anti-corruption layer. Give every coexistence mechanism a retirement batch.
6. Run a pilot: one isolated, valuable, loosely coupled, testable batch through the whole chain (rules, oracles, build, verification, cutover rehearsal). Exit when effort per registry line is measured and chain defects are fixed; then revise the strategy.
7. Industrialize only after the pilot exit: turn what worked into target standards, project skeletons, shared libraries, and prompts, each with an example and a test.
8. Accept big-bang only with a written justification, a rehearsed rollback, and the owner's signature. Record the decisions in `docs/governance/decision-log.md`.

## Outputs

- Primary artifacts: `docs/migration/strategy.md`, batch order with rationale, coexistence plan, pilot report, decision records.

## Guardrails

- No technology or architecture choice without the constraint it answers.
- Do not recreate the legacy structure in the target; describe the behavior, not the layout.
- Do not scale out before the pilot has measured results.
- Do not plan a big-bang cutover without justification and rollback.

## Handoff

`migration-batch-orchestration` to run the batches, `cutover-rollback` for the switch, `data-migration-reconciliation` for data.

## Provenance

Original to this suite; informed by the usual modernization trajectories (rehost, replatform, refactor, replace) and strangler-style coexistence (see `docs/references/legacy-method-gap-analysis.md`). Provenance is tracked in `docs/references/method-sources.md`.
