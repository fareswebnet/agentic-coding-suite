# legacy-stack-playbooks (compact)

When: Apply stack-specific playbooks (IBM i RPG/CL/DB2 and Adelia-style 4GL, COBOL mainframe, Oracle PL/SQL and Forms, PHP Symfony, Java target) for inventory sources, behavior traps, and oracle design.

Goal: Bring technology-specific knowledge to the generic migration skills without adding a skill per stack.

Steps:
1. Identify the legacy stacks and the target stack from the repository, the registry, and the brief.
2. Open only the matching reference: `references/ibm-i.md`, `references/ibm-i-4gl-adelia.md` (generated applications; read…
3. Inventory: use the reference's list of sources and extractors to feed `legacy-inventory-registry` and the probes of…
4. Rules: walk the reference's behavior traps while running `legacy-reverse-engineering`, and record each trap that applies as a…
5. Oracles: add the traps (numeric types, rounding, dates, text encoding, null handling, ordering, locking, restart) as boundary…
6. Target: apply the target-stack reference to the standards of `application-architecture` and `code-change-execution`.
7. If a stack has no reference, write one with the same sections (inventory sources, hidden behavior, security notes, oracle cases)…

Never:
- A reference is a checklist of where to look, not a fact about this system; confirm each trap on the real…
- Do not translate legacy structure line by line; use the target stack's idioms.
- Mark commands or behavior you could not verify on the actual release as unverified.

Next: `legacy-inventory-registry`, `legacy-reverse-engineering`, `characterization-testing`.
