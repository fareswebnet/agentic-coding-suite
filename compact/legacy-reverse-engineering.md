# legacy-reverse-engineering (compact)

When: Recover legacy behavior from code, data, and logs into a business-rule matrix with stable ids.

Goal: Turn implicit behavior into explicit, id-addressable rules that tests can prove and owners can approve.

Steps:
1. Work from the registry: process one batch of lines at a time and trace each from entry point through validation, business logic…
2. Extract rules as `RG-<SOURCE>-<NNN>` entries: condition, effect, inputs, source file and line, owner question, and risk class (R1…
3. Capture data lifecycle: tables touched, state transitions, events emitted, jobs and schedules, external calls, and files produced.
4. Mine tests, fixtures, production data shapes, logs, and support tickets for behavior the code hides; label each rule as observed…
5. Record bugs and accidental behavior separately as `LEGACY_DEFECT` with a decision: reproduce only if the business depends on it…
6. List unknowns and contradictions as questions with an owner; they block the batch only if they affect an R1 rule.
7. Record the legacy security baseline apart from the rules: authorization model, service accounts, adopted or shared privileges…
8. Link each rule to registry keys and, later, to scenarios and tests so `traceability-ledger` can check coverage.

Never:
- Mark each rule NEEDS_BUSINESS_VALIDATION until an owner confirms it; a rule in that state is not implemented…
- Do not read the legacy's intent into code; mark inferred rules as inferred until confirmed.
- Every R1 rule needs a second reader or an executed oracle before it is approved.
- Do not copy legacy defects by default; record the decision to improve or reproduce.
- Keep secrets and customer data out of extracted fixtures.

Next: `characterization-testing` to prove the rules, `bdd-specifications` to express them as scenarios, `legacy-stack-playbooks` for the stack's hidden-behavior…
