# legacy-migration

- Use for rewrites, replatforming, framework upgrades, or modernization of a system with thin documentation.
- Start with `legacy-usage-evidence` and `migration-strategy` (pilot batch first), then `legacy-inventory-registry`; use `legacy-stack-playbooks` for the named technologies; run `harnesses/legacy-migration.yaml`.
- Raise depth for `characterization-testing`, `independent-verification`, `data-migration-reconciliation`, and `cutover-rollback`.
- Measure progress only with `python scripts/sdlc_registry.py report`; classify R1 (money, security, compliance), R2 (workflows), R3 (presentation) and scale proof to risk.
- Reject legacy defects in the target by default and record each deliberate reproduction.

## Kickoff prompt

```text
You orchestrate the modernization of <system> to <target>, following harnesses/legacy-migration.yaml.
Start by understanding scope before proposing code: list the sources you can read, the questions for the business,
the usage probes to deploy, and the pilot candidate with the reasons. Label every statement as observed, hypothesis,
or decision. Do not write target code until the batch's rules are confirmed and its oracles run against the legacy.
```
