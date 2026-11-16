---
name: legacy-stack-playbooks
description: Apply stack-specific playbooks (IBM i RPG/CL/DB2 and Adelia-style 4GL, COBOL mainframe, Oracle PL/SQL and Forms, PHP Symfony, Java target) for inventory sources, behavior traps, and oracle design. Use when a legacy or target technology is named in a migration.
---

# SDLC Legacy Stack Playbooks

## Purpose

Bring technology-specific knowledge to the generic migration skills without adding a skill per stack.

## Workflow

1. Identify the legacy stacks and the target stack from the repository, the registry, and the brief.
2. Open only the matching reference: `references/ibm-i.md`, `references/ibm-i-4gl-adelia.md` (generated applications; read `ibm-i.md` too), `references/cobol-mainframe.md`, `references/oracle-plsql-forms.md`, `references/php-symfony.md`, `references/java-target.md`.
3. Inventory: use the reference's list of sources and extractors to feed `legacy-inventory-registry` and the probes of `legacy-usage-evidence`; check each command exists on the installed release.
4. Rules: walk the reference's behavior traps while running `legacy-reverse-engineering`, and record each trap that applies as a rule or a question.
5. Oracles: add the traps (numeric types, rounding, dates, text encoding, null handling, ordering, locking, restart) as boundary cases in `characterization-testing`, with the contractual rounding written down.
6. Target: apply the target-stack reference to the standards of `application-architecture` and `code-change-execution`.
7. If a stack has no reference, write one with the same sections (inventory sources, hidden behavior, security notes, oracle cases) and mark it unvalidated until an expert reviews it.

## Outputs

- Primary artifacts: stack-specific entries in the registry and rule matrix, trap checklist with outcomes, boundary cases for the oracle suite.

## Guardrails

- A reference is a checklist of where to look, not a fact about this system; confirm each trap on the real system.
- Do not translate legacy structure line by line; use the target stack's idioms.
- Mark commands or behavior you could not verify on the actual release as unverified.

## Handoff

`legacy-inventory-registry`, `legacy-reverse-engineering`, `characterization-testing`.

## Provenance

Original to this suite, written from general platform knowledge; the references are unvalidated against a live system until reviewed by a stack expert. Provenance is tracked in `docs/references/method-sources.md`.
