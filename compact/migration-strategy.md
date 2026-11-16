# migration-strategy (compact)

When: Choose and justify a modernization strategy (uplift, transform, reimagine, rehost, replace), order batches by a risk ladder, design the coexistence period, run a pilot batch, then industrialize.

Goal: Make the approach a documented decision tied to constraints, and prove it on a pilot before scaling it.

Steps:
1. Write the drivers: availability, volumes, regulation, team skills, budget and deadline, appetite for business change, market…
2. Compare the five trajectories against the drivers, with a rejection reason for each one dropped: uplift (newer version of the…
3. Justify the target architecture by constraint, not fashion; use `architecture-rfc` and `application-architecture`.
4. Order the batches by risk ladder: consultation and reporting, low-criticality workflows, stable interfaces, non-critical batch…
5. Design coexistence: the source of truth per entity during the transition, sync direction, frozen interface contracts, routing…
6. Run a pilot: one isolated, valuable, loosely coupled, testable batch through the whole chain (rules, oracles, build…
7. Industrialize only after the pilot exit: turn what worked into target standards, project skeletons, shared libraries, and…
8. Accept big-bang only with a written justification, a rehearsed rollback, and the owner's signature.

Never:
- No technology or architecture choice without the constraint it answers.
- Do not recreate the legacy structure in the target; describe the behavior, not the layout.
- Do not scale out before the pilot has measured results.
- Do not plan a big-bang cutover without justification and rollback.

Next: `migration-batch-orchestration` to run the batches, `cutover-rollback` for the switch, `data-migration-reconciliation` for data.
