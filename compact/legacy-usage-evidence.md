# legacy-usage-evidence (compact)

When: Gather runtime evidence of what a legacy system really uses (usage probes, call counters, job and log mining), flag dead or duplicated components, and drive keep, fix, abandon, or defer decisions with business sign-off.

Goal: Shrink the migration to what is alive before any rule is extracted or any code is written, with evidence a business owner can sign.

Steps:
1. Label every statement in the artifacts as observed (a fact with its source), hypothesis, or decision (who, when).
2. List the entry points per source: menus and screens, schedulers and batch calendars, inbound files and messages, partner…
3. Define usage probes: call counters per entry point, access logging on data objects, mining of job histories and logs, and any…
4. Deploy probes read-only and with low overhead, with the legacy owner's approval; keep personal data out of traces.
5. Flag candidates: no reference in the graph, zero use in the window, near-duplicates, superseded versions, orphaned data objects.
6. Rank by usage and criticality, and propose a disposition per candidate: port, fix first, replace, abandon, or defer.
7. Feed the result into `legacy-inventory-registry`: an ABANDONED line carries its evidence and signer; usage counts help order the…
8. Keep the probes running until cutover: a zero-call window on the old system after cutover is the evidence that decommissioning is…

Never:
- Never declare a component unused without evidence from a window that covers its cycle and an owner's…
- Absence in the static graph is a hypothesis, not proof.
- Probes never change legacy behavior or data, and never run in production without approval.
- Do not store personal or customer data in probe output.

Next: `legacy-inventory-registry` to freeze the denominator, `migration-strategy` to order the work.
