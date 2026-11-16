---
name: legacy-usage-evidence
description: Gather runtime evidence of what a legacy system really uses (usage probes, call counters, job and log mining), flag dead or duplicated components, and drive keep, fix, abandon, or defer decisions with business sign-off. Use before freezing a migration registry or when sizing a migration.
---

# SDLC Legacy Usage Evidence

## Purpose

Shrink the migration to what is alive before any rule is extracted or any code is written, with evidence a business owner can sign.

## Workflow

1. Label every statement in the artifacts as observed (a fact with its source), hypothesis, or decision (who, when). Never mix the three.
2. List the entry points per source: menus and screens, schedulers and batch calendars, inbound files and messages, partner interfaces, reports and printouts, ad-hoc queries by users. Add a static call and data-access graph: who calls whom, who reads or writes each data object.
3. Define usage probes: call counters per entry point, access logging on data objects, mining of job histories and logs, and any native usage statistics the platform already keeps. Choose an observation window that covers the business cycles (month end, quarter end, year end, seasonal runs).
4. Deploy probes read-only and with low overhead, with the legacy owner's approval; keep personal data out of traces.
5. Flag candidates: no reference in the graph, zero use in the window, near-duplicates, superseded versions, orphaned data objects. Check for calls the graph cannot see (name resolved at run time from a table or parameter, scheduled-once jobs, manual procedures).
6. Rank by usage and criticality, and propose a disposition per candidate: port, fix first, replace, abandon, or defer. The business owner decides; record the decision with date, owner, and evidence reference.
7. Feed the result into `legacy-inventory-registry`: an ABANDONED line carries its evidence and signer; usage counts help order the batches.
8. Keep the probes running until cutover: a zero-call window on the old system after cutover is the evidence that decommissioning is safe.

## Outputs

- Primary artifacts: `docs/migration/usage-evidence.md`, probe definitions and windows, decommission-candidate list with signers, hypotheses still open.

## Guardrails

- Never declare a component unused without evidence from a window that covers its cycle and an owner's signature.
- Absence in the static graph is a hypothesis, not proof.
- Probes never change legacy behavior or data, and never run in production without approval.
- Do not store personal or customer data in probe output.

## Handoff

`legacy-inventory-registry` to freeze the denominator, `migration-strategy` to order the work.

## Provenance

Original to this suite; informed by usage-based rationalization practice in modernization programs (see `docs/references/legacy-method-gap-analysis.md`). Provenance is tracked in `docs/references/method-sources.md`.
