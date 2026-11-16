# Skill evaluation: sonnet

Date: 2026-11-16  Runs per scenario: 3  Skills: compact

| Skill | Scenario | Baseline pass | With skill pass | Failures with skill |
|---|---|---|---|---|
| legacy-stack-playbooks | stack-adelia-generated | 0/3 | 2/3 | missing /(?i)(hand.edit|custom|diff|compare)/ |
| legacy-stack-playbooks | stack-symfony-hidden-rules | 3/3 | 3/3 | - |
| legacy-stack-playbooks | stack-rpg-rounding | 3/3 | 3/3 | - |
| legacy-usage-evidence | usage-dead-code-claim | 3/3 | 3/3 | - |
| legacy-usage-evidence | usage-short-window | 3/3 | 3/3 | - |
| migration-strategy | strategy-bigbang-microservices | 0/3 | 3/3 | - |
| migration-strategy | strategy-order | 3/3 | 3/3 | - |

Total: baseline 15/21, with skill 20/21.

String checks are a floor, not a quality judgement; read the replies for scenarios that fail.

## Replies

### legacy-stack-playbooks / stack-adelia-generated / baseline / FAIL

Reading the generated RPG is a useful input, but I wouldn't make it the primary source for the inventory. Use three sources and reconcile them.

## Why generated RPG alone falls short

- **It isn't the source.** Adelia regenerates it from its own repository, so anything you learn or annotate in the RPG is lost on the next generation. The migration decisions need to be made against the Adelia source.
- **It hides business intent.** Generated code has machine-made structure, names, and indicators. Screen flow, data dictionary semantics, reusable components, and the original developer's logic are much clearer in the Adelia model.
- **It's noisy.** Much of the volume is runtime plumbing and boilerplate, which inflates line counts and complexity metrics and misleads effort estimates.
- **It may not match what's in production.** Libraries can contain stale objects, source members that don't correspond to the running version, or hand-modified code.

## Recommended approach

1. **Inventory the Adelia repository (logical view)**
   - Export or list all programs, screens, reports, functions, data structures, files, and dictionary entries from the Adelia development environment.
   - Capture the call graph, file usage (read/update/write), and shared components using the Adelia cross-reference tools.
   - Record the Adelia version and generation options, since these affect the target RPG and runtime dependencies.

2. **Inventory the system (runtime view)**
   - Run `DSPOBJD`, `DSPPGMREF`, `DSPFD`, and `DSPDBR` on the production libraries to get objects, program-to-file references, and dependencies. Wrap them in outfiles and query them with SQL.
   - Use the SQL catalog services (`QSYS2.OBJECT_STATISTICS`, `SYSTABLES`, `SYSCOLUMNS`, and so on) for scripted extraction.
   - Collect usage data (last-used date, use count) from object descriptions, or from journals, audit data, and job logs, to separate live from dead code.
   - Include job schedules, subsystems and job descriptions, data areas, data queues, user spaces, output queues, and CL programs that launch or glue the application together.

3. **Use the generated RPG selectively**
   - Match each generated object back to its Adelia source member to confirm that the deployed version corresponds to the repository.
   - Look for hand-written or modified RPG, CL, and SQL that the Adelia repository doesn't know about. These are the highest migration risk.
   - Use it to check specific behaviors, such as rounding, date handling, or locking, where you need the actual runtime semantics.

4. **Reconcile the two views**
   - Flag items that exist only in the repository (unused or never deployed) and items that exist only on the machine (orphans or hand-built).
   - Classify each item as migrate, retire, rewrite, or replace.

5. **Capture non-code dependencies**
   - Adelia runtime service programs and licensing, external interfaces (APIs, FTP, MQ, EDI), printing and spool handling, authorities, and batch dependencies.

## Practical advice

- Ask whether the Adelia source repository is complete and current. If it is, it is your system of record. If it has gaps, the generated RPG becomes a fallback, and you should document those gaps explicitly.
- Decide the migration strategy first (stay on IBM i and modernize, convert to another language, or replace). It determines how much detail you need from each source.
- Consider contacting the Adelia vendor (Aubay) about export formats and tooling. They may offer repository export or migration utilities that give you a structured model rather than text to parse.

If you can share which pieces you have access to (repository export, production libraries, documentation), I can help draft the specific queries and a reconciliation template.

### legacy-stack-playbooks / stack-adelia-generated / with skill / PASS

# Short answer

Don't make the generated RPG the primary inventory source. Use it as a second, cross-checking source. In a generated application, the Adelia definitions (the 4GL source and its repository/dictionary) hold the intent, and the generated RPG is a build artifact full of generator boilerplate. Reading it line by line is slow, and it hides the business rules. But what runs on the machine is the compiled generated code, so you also need it to detect drift and to build oracles.

I have no repository or tool access here and haven't opened the Adelia reference, so what follows is a checklist of where to look. It is not a fact about your system, and every command or behavior is **unverified** until confirmed on your actual release.

## Why "just read the generated RPG" is risky

- **Noise:** generator-produced RPG is verbose and structurally repetitive, so the rules are buried in it.
- **Regeneration:** any manual reading is lost value if the source is regenerated, and any manual patch to generated code is a hidden behavior you need to find.
- **Drift:** the deployed objects may not match the current Adelia source (older generation, different generator options or version, hand-edited members, objects compiled from another library).
- **Missing context:** a lot of behavior lives outside the program, such as the library list, file overrides, data areas, job descriptions, and the runtime or services the generated code calls.

## Proposed approach

1. **Identify stacks and target.** Adelia-generated RPG, CL, DB2 for i (DDS and/or SQL), possibly hand-written RPG. I also need to know the target stack, since it drives the later steps.
2. **Inventory from several sources** and feed them into the inventory registry:
   - **Adelia repository/source:** screens, functions, processes, batch programs, data dictionary and file definitions, generation options, and the generator version in use.
   - **Deployed objects on IBM i:** object lists and descriptions per library (for example `DSPOBJD`), program-to-object references (`DSPPGMREF`), service programs and binding directories, and source or compile attributes showing which generation produced each object. Verify the exact commands on your release.
   - **Database:** physical and logical files, SQL views and indexes, keys and access paths, triggers, constraints, journaling and commit control.
   - **CL and operations:** CL programs, job schedule entries, job descriptions, subsystem and routing setup, data areas and data queues, message files, output queues, spool usage.
   - **Security:** authorities, adopted authority, user profiles, and menu-level access.
   - **Generated vs. source reconciliation:** list the objects with no Adelia source, the sources with no deployed object, and the objects that differ from a fresh regeneration (if you can regenerate in a safe environment).
3. **Reverse-engineer rules from the Adelia source first.** Use the generated RPG only for questions the source can't answer, such as what a generator option actually does at runtime.
4. **Walk the behavior traps and record each one that applies as a finding.** Confirm each on the real system:
   - packed/zoned decimal handling, field sizes, and truncation
   - rounding (half-adjust or not) and intermediate precision
   - date and time formats, and the meaning of zero or blank dates
   - CCSID and EBCDIC conversion, trailing blanks, case
   - null handling versus blank/zero defaults
   - result ordering that depends on key or access path rather than an explicit sort
   - record locking and commit behavior
   - batch restart and rerun behavior
   - dependencies on library list, overrides (`OVRDBF`), the LDA, and job attributes
   - generator-runtime specifics (screen/session handling, parameter passing, message handling), which I can't confirm without the reference
5. **Build oracles:** capture inputs and outputs from the running system (database state before and after, spool output, messages, return codes) with boundary cases for each trap above. These feed characterization testing.
6. **Don't translate structure line by line.** The target should use its own idioms, not mimic the generated program layout.

## Questions I need answered

- What is the target stack?
- Do you have the Adelia repository and the generator version, and can you regenerate in a non-production environment?
- Is there hand-modified generated code, or objects with no known source?
- Can you take representative production data and job runs for the oracles?

With those answers, the next step is to set up the inventory registry and the first reverse-engineering and characterization passes.

### legacy-stack-playbooks / stack-adelia-generated / baseline / FAIL

I'd push back on making generated RPG the primary source. It's useful, but as a cross-check rather than the foundation.

## Why generated RPG is a poor primary source

- **The logic is buried.** Generated code is mostly scaffolding: screen handling, runtime calls, and mechanical naming. The business intent that the Adelia developer wrote is hard to recover from it.
- **It's a build artifact.** Whatever you migrate or maintain in the future will derive from the Adelia source, so the inventory should describe that.
- **Volume is inflated.** One Adelia function can expand into hundreds or thousands of RPG lines, so effort estimates based on RPG line counts will be misleading.
- **Dependencies hide in the runtime.** Calls through the Adelia runtime or dynamic calls may not show up in a plain read of the RPG.

Where the RPG does help is in confirming what is actually deployed, finding drift between source and objects, and serving as a fallback for any program whose Adelia source is missing.

## Suggested approach

1. **Inventory the objects on the system (ground truth of what exists).**
   - List libraries, programs, service programs, files, data areas, data queues, commands, and binding directories. `QSYS2.OBJECT_STATISTICS` and `DSPOBJD` work for this.
   - Capture creation date, last-changed date, last-used date, and use count for each object.
   - Use `DSPPGMREF`, `DSPDBR`, and `QSYS2.PROGRAM_INFO` or `BOUND_MODULE_INFO` to map program-to-file and program-to-program references.

2. **Inventory the Adelia source (the real source of truth).**
   - Export or list everything from the Adelia repository or dictionary: programs, functions, screens, reports, file and field definitions, and shared components.
   - Check the vendor documentation or support for repository-query or export tools. I can't verify the specifics of your version, so confirm what's available.

3. **Reconcile the two.**
   - For each deployed object, check whether it has Adelia source and whether the source timestamp is consistent with the object's creation date.
   - Flag orphans in both directions: objects without source, and source without a deployed object.
   - Flag hand-written RPG, CL, and other non-Adelia code, which needs a separate analysis path.

4. **Add runtime usage evidence.**
   - Use last-used dates and counts, job scheduler entries, job logs, and journal or audit data to separate live programs from dead ones. Dead code is often 20–40% of a legacy estate, and every program you can drop saves migration effort.

5. **Capture the data and interfaces.**
   - Record physical and logical files, DDS or SQL definitions, and the Adelia dictionary's field semantics.
   - Record external touchpoints: spool output, FTP, data queues, web services, EDI, scheduled jobs, and any exit programs.

6. **Classify and prioritize.**
   - For each component, record its business function, complexity, dependencies, usage, and migration disposition (rewrite, retain, retire, replace).

7. **Use generated RPG selectively.**
   - Read it for cases the Adelia source can't explain: suspected drift, missing source, or debugging runtime behavior.
   - Static-analysis tools can run on the RPG for metrics and call graphs. Treat the results as supplementary.

## Questions to settle early

- What is the migration target: staying on IBM i, moving to another platform, or replacing the application? That determines how much business logic you need to extract and in what form.
- Is the Adelia repository complete and current? If not, the generated RPG becomes much more important as a recovery path.
- Does the vendor provide migration or conversion tooling, or professional services for this?

In short, use the Adelia repository for what the application is meant to do, system object data for what exists and is used, and generated RPG to verify the gaps.

### legacy-stack-playbooks / stack-adelia-generated / with skill / PASS

# Short answer

Don't make the generated RPG the only source. It is what runs, so it is a good ground truth for checking behavior, but it is a poor place to find intent, and it hides the real structure of an Adelia application. Inventory three layers and reconcile them.

I have no repository or machine access here, and I haven't opened the Adelia reference. Everything below is a checklist of where to look, not a fact about your system. Confirm each item on your actual release and mark anything you can't verify as unverified.

## Why RPG alone is risky

- **Generated code is not the design.** Business rules live in the Adelia model (dictionary, screens, functions, actions, and so on). In the RPG they are buried under framework and runtime-call boilerplate, so you can't tell a rule from plumbing.
- **Source and objects can drift.** Programs on the machine may have been generated from an older model, patched, or regenerated with different options. You need to know which model version produced each object.
- **Some behavior isn't in any one program.** It comes from the runtime or framework, the library list, data areas, job and CL setup, authorities, and the scheduler.
- **Volume.** Generated RPG is verbose, so reading it by hand is slow and error-prone.

## How to proceed

**1. Identify the stacks and the target.** Confirm the following with the team:
- Adelia version and runtime.
- Whether the generated code is RPG III or RPG IV.
- Any hand-written RPG, CL, or SQL alongside the generated code.
- The target stack.

**2. Build the inventory in layers and feed it to the registry.**
- **Adelia repository (the intent layer).** Export or list the models, dictionary, screens, functions, actions, and generation parameters. Record which generation produced which object.
- **Generated objects (the runtime layer).** Per library, list programs, service programs, display and printer files, and binding directories. Typical sources are `DSPOBJD`, `DSPPGMREF`, the `QSYS2` catalog views, and the compile or source attributes. Verify the exact commands on your release.
- **Data layer.** Physical and logical files and any SQL objects (`DSPFD`, `DSPFFD`, `DSPDBR`). Capture keys, field types, and the dependencies between files.
- **Execution environment.** CL programs, job descriptions, library lists, data areas, `*LDA` usage, message queues, output queues, and the job scheduler. Also capture user profiles and authorities, including adopted authority, plus triggers, journaling, and commitment control.
- **Dynamic calls and unreferenced objects.** Static cross-references miss calls by name from a variable. Cross-check against actual usage, such as last-used dates or job logs.

**3. Reverse-engineer rules from the model first, and use the RPG to check them.** For each rule, record where it was seen (model, RPG, or both). Where the model and the generated code disagree, treat the running code as the oracle and log the discrepancy.

**4. Walk the behavior traps.** Record each one that actually applies, after confirming it on real data and code:
- Packed and zoned decimals, and their sign handling.
- Rounding (half-adjust versus truncation) in intermediate and final results.
- Dates stored as numbers, with special values such as 0 or 99999999.
- CCSID and EBCDIC conversion, and trailing-blank semantics.
- Blank versus null, and default values.
- Sort order and collating sequence, and key order versus arrival order.
- Record locking and commitment control.
- Restart and recovery behavior in batch jobs.
- Indicators and screen-flow state.

**5. Turn the traps into oracle cases.** Create boundary cases for numeric types, rounding, dates, encoding, nulls, ordering, locking, and restart. Capture expected outputs from the legacy system with the characterization tests, not from your reading of the code.

**6. Target.** Rebuild with the target stack's idioms. Don't translate the generated RPG line by line, because that would carry the generator's structure into the new system.

## Reply to the team

Reading the generated RPG is useful as a verification step and as the source of truth for behavior. It should not be the starting point of the inventory. Start from the Adelia repository and the object, data, and job inventory, then use the RPG and real runs to confirm what the rules actually do.

If you can share the Adelia version, the target stack, and a list of libraries, the next step is to set up the inventory registry and the first set of probes.

### legacy-stack-playbooks / stack-adelia-generated / baseline / FAIL

# Inventorying an Adelia application for migration

**Short answer:** don't make the generated RPG your primary source. Use it as a cross-check on what's deployed, and build the inventory from the Adelia repository plus system-level metadata and runtime usage data.

## Why generated RPG is the wrong primary source

- **It's compiler output, not the source of truth.** Business intent lives in the Adelia source: the definitions, dictionary, screens and logic. Generated RPG is full of runtime scaffolding, machine-chosen names and expanded boilerplate, so the logic is hard to recognize.
- **It's noisy and large.** Reading it is slow and error-prone, and it hides the structure (screens, reports, data model, reuse) that Adelia already describes explicitly.
- **It can drift.** The deployed object may not match the current Adelia source. Look for stale generations, objects patched on the machine, or source that was never regenerated.
- **It misses things.** Dictionary-level rules, shared definitions and generation parameters often don't show up clearly in the RPG.

What the RPG *is* good for is evidence of what actually runs. Use it to verify, to find hand-modified objects, and as a fallback if some Adelia source is lost.

## Recommended approach

1. **Define scope and sources.** List the libraries, applications, environments (dev, test, prod) and the Adelia repository or repositories involved.

2. **Inventory the Adelia repository (the logical view).**
   - Programs, screens, reports, batch jobs, functions and shared components.
   - The data dictionary, file and field definitions, and the relationships between them.
   - Cross-references and impact analysis. Adelia's tooling should have these, and the repository is stored in Db2 for i files, so you can often query it with SQL. Confirm the details with your Adelia version's documentation or the vendor.

3. **Inventory the machine (the physical view).**
   - Objects: `DSPOBJD` or the `QSYS2.OBJECT_STATISTICS` service, covering programs, service programs, files, data areas, data queues, commands and job descriptions.
   - Dependencies: `DSPPGMREF`, `DSPDBR`, and the `QSYS2` catalog views for tables, views, indexes, constraints and triggers.
   - Scheduling and integration: job schedule entries or any scheduler, CL programs, FTP and IFS usage, web services, EDI, printers and spool flows.
   - Security: authorities, adopted authority, user profiles and menus.

4. **Reconcile the two views.**
   - Match each deployed object to its Adelia source and generation timestamp.
   - Flag orphans (deployed with no source), unreferenced source, and mismatches. These are your risk list.
   - Identify the non-Adelia parts: hand-written RPG, CL, COBOL, SQL, and third-party packages.

5. **Measure actual usage.**
   - Last-used date and use count from object statistics, plus object auditing (`QAUDJRN`) or job logs, ideally across a full business cycle including month-end and year-end.
   - This separates live code from dead code, which can reduce migration scope considerably.

6. **Extract the data model and business rules.** Use the dictionary for entities, keys and validations. Document calculation and decision logic from the Adelia source, and from generated RPG only where the source is missing or in doubt.

7. **Classify and plan.** For each component record its type, owner, complexity, usage, dependencies and interfaces, then pick a disposition: retire, rehost, refactor or rewrite. The choice of target (staying on IBM i, a modern language, or an Adelia modernization path) affects how deep the rule extraction needs to go.

## Deliverables

- A component catalog (object, type, source, last generation, last used, owner).
- A dependency graph covering programs, files and external interfaces.
- A data model and dictionary export.
- A list of discrepancies (orphans, drift, manual patches).
- A usage report separating live from dead code.
- A migration-disposition matrix with risks.

## Practical notes

- If you do analyze generated RPG, do it with tooling (parsers, static analysis, scripts), not manual reading, and tie the results back to Adelia objects.
- Take a snapshot or save of the libraries and repository before you start, so the inventory reflects a fixed point in time.
- Involve the Adelia vendor or an in-house expert early to confirm which repository and export facilities your version supports.

### legacy-stack-playbooks / stack-adelia-generated / with skill / FAIL

# Short answer

Don't make the generated RPG the primary inventory. It is evidence of what runs, but not the source of the logic. Inventory both layers and reconcile them.

I can't open `references/ibm-i-4gl-adelia.md` in this exercise, so what follows is general 4GL and IBM i practice. Adelia-specific names, commands and behaviors are **unverified** until confirmed on your release.

## Why generated RPG alone is a trap

- **It is output, not intent.** The business rules live in the 4GL definitions (dictionary, screens, functions, programs). Generated code buries them in boilerplate and runtime scaffolding, and it is hard to tell a rule from framework plumbing.
- **It can drift from the 4GL source.** Members may be stale, regenerated, hand-patched on the machine, or left over from retired versions. What is compiled in production may match neither the source members nor the 4GL repository.
- **Migrating from it reproduces the generator's structure.** Translating that line by line into the target stack gives you unidiomatic code, which we want to avoid.
- **It still matters.** It is the closest thing to the runtime truth, so it works as a cross-check and as a source of hidden behavior.

## Proposed approach

1. **Confirm the stack and target.** Establish the Adelia version, the generated language (RPG IV or older RPG/CL), the DB2 for i usage, and the target stack from the brief.
2. **Inventory the 4GL repository first** (feeds `legacy-inventory-registry`):
   - Dictionary and data definitions, screens/dialogs, functions, programs, batch jobs, and their dependencies.
   - Export or listing facilities of the Adelia development environment. Which ones exist on your release is unverified.
3. **Inventory the machine** as the second layer:
   - Libraries and objects: programs, service programs, files, data areas, data queues, job descriptions, and the job scheduler entries.
   - Generated source members and compiled-object metadata (compile dates, source-to-object link).
   - DB2 for i: physical and logical files, DDS vs SQL definitions, constraints, triggers, journaling.
   - CL programs, menus, authority and adopted-authority settings, and the runtime or library list setup.
4. **Reconcile the layers.** For each production object, find its 4GL origin and flag these:
   - Objects with no 4GL source.
   - 4GL elements that were never generated or deployed.
   - Compile dates that are newer than the 4GL, or members that were edited by hand.
   - Dead or duplicate objects. Check actual usage, not just presence.
5. **Reverse-engineer rules from the 4GL, and verify them in the generated code and data.** Record each trap that applies as a finding, and confirm it on the real system rather than assuming it. Traps to check:
   - Packed and zoned decimal handling, and rounding or truncation (half-adjust behavior).
   - Date formats and "zero dates".
   - EBCDIC and CCSID conversion, and trailing-blank and fixed-length semantics.
   - Null versus default values.
   - Sort order and collating sequence.
   - Record locking and commitment control.
   - Restart and recovery in batch jobs.
   - Implicit logic from the runtime (indicators, message handling, default validations).
6. **Build oracles** (`characterization-testing`). Capture inputs and outputs from the running system, including the boundary cases from the trap list: numeric extremes, rounding, dates, text encoding, nulls, ordering, locking and restart. Run batch jobs and screens against known data so the oracles reflect production behavior, not just the code.
7. **Apply target idioms.** Design the target with the target stack's conventions (`application-architecture`, `code-change-execution`), not by transliterating generated RPG.

## Practical advice for the team

- Treat reading RPG as a targeted tool: use it to settle a specific question about behavior that the 4GL doesn't answer, or to detect drift.
- Ask early whether the 4GL repository is complete and current, and whether anyone patches the generated code directly. That answer decides how much weight each layer gets.
- Keep unverified items (commands, export features, release-specific behavior) marked as such in the registry until someone has run them on your actual system.

Next skills: `legacy-inventory-registry`, `legacy-reverse-engineering`, `characterization-testing`.

### legacy-stack-playbooks / stack-symfony-hidden-rules / baseline / PASS

**First, check that the docs match what the code does.** Docs describe intent and drift over time. In a Symfony app, much of the real behavior lives outside controllers and entities, so porting only those, with tests written from the docs, would likely miss critical logic and bake doc errors into your tests.

## 1. Verify docs against behavior
- Pick the money-critical flows (cart, checkout, payment, order status, refunds) and trace them in the code. Compare the result to the docs.
- Write **characterization tests** against the running legacy app (recorded requests and responses, DB state before and after). Use these as the source of truth, and use the docs as a secondary reference.

## 2. Find logic hiding outside controllers and entities
- **Doctrine lifecycle callbacks and listeners/subscribers** (`prePersist`, `preUpdate`, `postLoad`). These often set timestamps, slugs, totals, stock, and order numbers.
- **Event subscribers/listeners** (`kernel.request`, `kernel.response`, security events, custom domain events). They can change carts, locale, currency, and redirects invisibly.
- **Services and DI config** (`services.yaml`, decorators, compiler passes, tagged services). Pricing, tax, discount, shipping, and stock rules usually live here.
- **Repositories and custom DQL/SQL.** Query behavior (filters, sorting, soft deletes, default scopes) is part of the contract.
- **Form types, validation constraints (annotations, YAML, XML), and data transformers.** These define input rules and error messages.
- **Security config:** firewalls, `access_control`, role hierarchy, voters, `@IsGranted` or `@Security` annotations, CSRF, remember-me, and password encoders. Hashing needs a migration plan so existing users can still log in.
- **Console commands, cron jobs, and Messenger handlers.** These cover emails, stock sync, abandoned carts, and exports, and may never appear in controller docs.
- **Twig templates and extensions.** Price formatting, rounding, and business logic often leak in there.

## 3. Check the data layer
- Compare the **actual DB schema** (and migrations) with the entity mappings. Look for drift, extra columns, triggers, views, and indexes.
- Check precision and rounding for money (decimal vs float, currency handling, tax-inclusive vs exclusive).
- Check enum-like values, serialized or JSON columns, and inheritance mappings. Note any Doctrine-specific behavior such as cascades, orphan removal, and optimistic locking.
- Plan data migration and cutover. IDs, order numbers, and password hashes must stay compatible.

## 4. Inventory external integrations and bundles
- Payment gateways, webhooks and their idempotency, shipping and tax providers, email, search, and storage.
- Third-party bundles (e.g. FOSUserBundle, Sonata, API Platform, Liip) that provide behavior you'll need to replace.
- Environment config, secrets, and feature flags.

## 5. Capture the public contract
- Routes (including redirects and legacy URLs for SEO), HTTP methods, status codes, headers, cookies and session usage (the cart often lives in the session), and serialization formats.
- Locale, currency, and i18n handling.

## 6. Settle test strategy before porting
- Run the same tests against both the old and new apps, so parity is measurable.
- Prioritize by risk: payments and orders, then pricing and discounts, then auth, then catalog and content.
- Check test coverage and CI in the legacy app. Existing tests are another source of truth.

## Suggested order
1. Make a quick inventory of listeners, subscribers, services, commands, and bundles.
2. Spot-check the docs against the code on 2-3 critical flows to estimate how reliable they are.
3. Build characterization tests for those flows.
4. Compare the schema and plan the data migration.
5. Port by feature slice (entity, logic, controller, tests together), not layer by layer.

If the spot-check shows the docs are mostly accurate, you can lean on them more. If not, treat them only as a map.

### legacy-stack-playbooks / stack-symfony-hidden-rules / with skill / PASS

# What to check first

**Don't treat the docs as the oracle.** Documented controllers and entities describe intended behavior. In a Symfony e-commerce app, much of the behavior that matters (prices, stock, taxes, order state, access) happens outside them. Check these in order.

## 1. Inventory what the docs don't cover
Build a list of every behavior source, not just controllers and entities, and mark each as documented or undocumented:
- **Event listeners and subscribers** (`kernel.request`, `kernel.response`, `kernel.exception`, custom domain events). They often apply discounts, locale, currency, or redirects.
- **Doctrine lifecycle callbacks and listeners** (`prePersist`, `preUpdate`, `postLoad`). They often set order numbers, totals, and timestamps.
- **Security:** firewalls, `access_control`, voters, guard authenticators, password encoder config, remember-me, and any admin bundle permissions.
- **Console commands and cron jobs:** stock sync, abandoned carts, exports, invoice generation, and cleanup.
- **Messenger or queue handlers, mailers, and webhooks** (payment provider callbacks, shipping).
- **Database-side logic:** triggers, views, stored procedures, sequences, and unique or check constraints that the entities don't reflect.
- **Twig templates and extensions** that compute or format prices, taxes, or conditions.
- **Forms and validators:** constraints and data transformers.
- **Third-party and overridden bundles** (admin, user management, API layer) and anything in `config/packages`, `services.yaml`, and `.env*` per environment.
- **Custom Doctrine types and DQL functions, and raw SQL in repositories.**

## 2. Check the highest-risk behavior traps
These are places to look, not facts about your system. Confirm each one against the real code and data.
- **Money:** float vs decimal storage, where rounding happens (per line or per total), tax-inclusive vs exclusive, and currency minor units.
- **Dates and time zones:** server vs DB vs user time zone, mutable `DateTime`, and DST edges for promotions and cutoffs.
- **Text and collation:** MySQL case-insensitive uniqueness (emails, SKUs, coupon codes), `utf8` vs `utf8mb4`, and trimming or normalization.
- **Null and default handling:** how nullable columns, empty strings, and missing form fields behave.
- **Ordering:** queries with no `ORDER BY` that rely on insertion order or index order.
- **Concurrency:** stock decrement, coupon redemption limits, and double-submit on checkout (locking, optimistic versioning, idempotency).
- **State machines:** order and payment status transitions, and which are allowed, in what order, and by whom.
- **Sessions:** cart storage (session vs DB), merge-on-login behavior, and expiry.
- **Number sequences:** order, invoice, and gap-free numbering.
- **Serialization:** exact JSON or XML shapes, field names, and null vs omitted fields if other systems consume them.

## 3. Make the running legacy system the oracle
- Write characterization tests against the **live legacy app** (or a copy with realistic data), not from the docs. Record actual inputs and outputs.
- Where docs and observed behavior disagree, log the difference and get an explicit decision on which to keep.
- Turn each trap above into boundary cases, such as rounding at .005, zero or negative quantities, expired promotions, two simultaneous checkouts, and a refund after a partial shipment.
- Use real, anonymized production data for pricing and tax cases, since edge cases usually live there.

## 4. Port using the target stack's idioms
Don't translate Symfony structure line by line. Listeners, Doctrine callbacks, and Twig logic should become explicit domain services or the target framework's equivalents. Map the security model deliberately, since Symfony 4 guard authenticators and encoders have no one-to-one equivalent elsewhere.

## Caveats
- I have no repository access, so everything above is a checklist, not a finding about your app.
- Symfony 4 is end-of-life, and I haven't verified its behaviors against your exact release and bundle versions. Treat version-specific details (security config, encoder defaults) as unverified until you confirm them.

If you can share the bundle list (`composer.json`), the `config/` tree, and a list of listeners and commands, I can help prioritize the inventory and draft the first set of characterization cases.

### legacy-stack-playbooks / stack-symfony-hidden-rules / baseline / PASS

**Check first: whether the docs match what the running app actually does.** Docs describe intent, and they drift. If you write tests from the docs and port to match them, you may faithfully reproduce the documentation instead of the behavior customers and the business rely on. Treat the legacy app as the source of truth and the docs as a map.

**1. Validate the docs against reality (cheap, high value)**
- Pick the 5-10 most revenue-critical flows (add to cart, checkout, payment, order confirmation, refunds, stock decrement, promo codes) and run them against the legacy app. Record real requests and responses, DB state changes, and emails sent.
- Compare these to the docs. Every mismatch tells you how much to trust the rest.
- Use these recordings as characterization (golden-master) tests that run against both the old and new stacks. They're more reliable than tests derived from prose.

**2. Find the logic that lives outside controllers and entities**
Symfony apps often hide important behavior in places that porting only controllers and entities will miss:
- **Event listeners/subscribers** (kernel events, custom domain events): often handle order status changes, emails, and stock updates.
- **Doctrine lifecycle callbacks and listeners** (`prePersist`, `preUpdate`, entity listeners): timestamps, slugs, computed totals, and audit trails.
- **Services** in `services.yaml`, including decorators and compiler passes, that change wiring in non-obvious ways.
- **Security config:** `security.yaml` firewalls and `access_control`, voters, and user providers. Authorization rules are frequently not in the controllers.
- **Form types and validation constraints** (annotations, YAML, or XML): these define the real input rules.
- **Twig templates and extensions:** price formatting, conditional display, and sometimes business logic.
- **Console commands and cron jobs:** abandoned-cart emails, feed exports, stock syncs, and cleanup tasks.
- **Messenger/queue handlers** and any async work.
- **Custom repositories and DQL/native SQL:** filtering, search, and reporting semantics.

**3. Money, tax, and numeric behavior**
- How prices are stored (decimal vs. integer cents vs. float), rounding mode and order of operations (per line vs. per order), currency handling, tax and shipping calculation, and discount stacking.
- Differences in decimal and rounding behavior between PHP and the target language are a classic source of subtle bugs. Write explicit test cases with edge values.

**4. Data model and migrations**
- Compare the real database schema (and Doctrine migrations history) against the entity docs. Production schemas often have columns, indexes, or constraints that entities don't describe.
- Decide whether the new stack uses the same database (data migration and compatibility) or a new one, and check for data quirks: nullable fields that are never null, legacy statuses, and orphaned rows.

**5. External integrations**
- Payment gateway, shipping, tax services, email, search, analytics, and webhooks. Check how callbacks and idempotency are handled, and which credentials and environments exist.
- Webhook handlers and return URLs are often undocumented but critical.

**6. Sessions, cart, and auth behavior**
- How the cart is persisted (session vs. DB), guest vs. logged-in merge behavior, password hashing algorithm (you'll need to support existing hashes or force resets), remember-me, CSRF, and rate limiting.

**7. Non-functional and implicit behavior**
- URL structure and redirects (SEO), locale handling, caching (HTTP cache, Doctrine cache), file uploads and media paths, error pages, and logging/monitoring that ops relies on.

**8. Dependencies and version risk**
- Symfony 4 is long out of support. Check which bundles (e.g., FOSUserBundle, Sonata, API Platform, custom forks) supply behavior that won't have a direct equivalent in the new stack.

**Suggested order**
1. Spot-check docs vs. behavior on critical flows.
2. Inventory the non-controller/entity logic (listeners, security, commands, templates, config).
3. Capture characterization tests and fixtures from the running app.
4. Pin down money/tax/rounding and schema details.
5. Then port, using the characterization tests as the acceptance gate and the docs as supporting context.

If you can share the repo later, a quick grep of `EventSubscriber`, `@ORM\HasLifecycleCallbacks`, `security.yaml`, `src/Command`, and `config/services.yaml` will show quickly how much hidden logic exists.

### legacy-stack-playbooks / stack-symfony-hidden-rules / with skill / PASS

# What to check first

**Whether the docs match what the app actually does.** Documentation describes intent. If you write tests from the docs, you only prove the port matches the docs. Treat the running legacy app as the oracle and the docs as hypotheses. Use the docs to decide where to look, then confirm each claim against the real system (or its recorded outputs) before it becomes a requirement.

## 1. Inventory what exists beyond controllers and entities

In a Symfony 4 app, much of the behavior lives outside those two layers. I can't see your repo, so this is a checklist of where to look, not a finding about your system:

- **Event listeners and subscribers**, including kernel events and Doctrine lifecycle callbacks (`prePersist`, `preUpdate`, `postLoad`). These often set totals, slugs, timestamps, stock, and status transitions invisibly.
- **Security**: `security.yaml` firewalls, `access_control`, role hierarchy, voters, password encoders, remember-me, and login success/failure handlers.
- **Forms and validation**: form types, data transformers, constraints (including validation groups and custom validators), and serializer config.
- **Console commands and cron jobs**: order expiry, cart cleanup, feeds, reports, and imports.
- **Messenger, queues, and mail**: async handlers, email templates, and retry behavior.
- **Twig**: price formatting, conditional display, and logic embedded in templates or Twig extensions.
- **Database-side logic**: triggers, views, stored procedures, sequences, collations, and indexes or unique constraints that the entities don't fully show.
- **Config and environment**: `.env` values, feature flags, parameters, and per-environment overrides.
- **Third-party bundles and integrations**: payment gateway, tax, shipping, search, and webhooks (including inbound signature checks and idempotency).
- **Vendor patches or overrides** of bundles.

Record each item with its source, owner, and whether it's confirmed or only claimed.

## 2. Behavior traps to confirm on the real system

Each of these is a candidate, not a known fact about your app:

- **Money**: float vs. decimal, Doctrine returning `decimal` as strings, rounding mode and when rounding happens (per line vs. per order), tax-inclusive vs. exclusive, discount and tax ordering, and currency handling.
- **Dates and time zones**: PHP default time zone vs. DB time zone, `DateTime` vs. `DateTimeImmutable`, and how "end of day" and expiry are computed.
- **Text and collation**: MySQL case-insensitive uniqueness (emails, SKUs, coupon codes), `utf8` vs. `utf8mb4`, trimming, and accent handling.
- **Null and default handling**: empty string vs. null, and defaults set in the DB vs. in code.
- **Ordering and pagination**: queries without an explicit `ORDER BY` that "happen" to return a stable order.
- **Concurrency**: stock decrement, locking (optimistic version fields or pessimistic locks), double-submit, and order number generation.
- **Cart and session**: session storage, cart merge on login, and guest-to-customer transitions.
- **Passwords**: hash algorithm and migration strategy (you can't re-hash without the plaintext, so the new stack must verify old hashes).
- **Restart and idempotency**: what happens when a payment webhook is retried, or a cron job runs twice or fails halfway.

## 3. Turn the traps into oracle cases

Before porting, capture characterization cases from the legacy system itself:

- Boundary values: zero, negative, max quantity, rounding edges like x.xx5, and empty or whitespace inputs.
- Real or anonymized production data samples, including old and odd records.
- Recorded request/response pairs, emails sent, DB state before and after, and queued messages for key flows (checkout, refund, coupon, stock change, login, password reset).
- Authorization matrix: role × endpoint × expected result.

Run the same cases against the new stack and diff the results. Anything the docs don't explain is a hidden rule to document or a deliberate difference to decide on.

## 4. Port by intent, not structure

Don't translate controllers and entities line by line. Use the target stack's idioms, and carry over only behavior you've confirmed. Where legacy behavior looks like a bug, log it as an explicit decision (keep, fix, or drop) rather than silently changing it.

## Suggested order

1. List the target stack and any other legacy components (the brief should say).
2. Build the inventory above and mark each doc claim as confirmed, contradicted, or unverified.
3. Run the trap checklist and record which ones apply.
4. Write characterization tests against the legacy app.
5. Then port, using the tests as the acceptance gate.

If you can share the controller list, `security.yaml`, the listeners/subscribers directory, and the DB schema, I can help build the inventory and the first set of oracle cases.

### legacy-stack-playbooks / stack-symfony-hidden-rules / baseline / PASS

**First, check whether the docs match what the app actually does.** For a port, the real risk is that the controllers and entities are documented, but much of the behavior lives elsewhere, and docs drift. I'd verify these in roughly this order.

## 1. Doc accuracy
- Compare the docs against the code for a few critical flows (checkout, pricing, order status changes). If they've drifted, tests written from the docs will encode the wrong behavior.
- Prefer **characterization tests**: run the legacy app, record real inputs and outputs (HTTP responses, DB state changes, emails sent), and use those as the oracle. Use the docs as a guide, not as the source of truth.

## 2. Behavior that lives outside controllers and entities
Symfony apps hide business logic in places a controller/entity port will miss:
- **Doctrine lifecycle callbacks and listeners/subscribers** (`prePersist`, `preUpdate`, timestamps, slug generation, stock decrement, total recalculation).
- **Event subscribers and listeners** on kernel events (locale, cart/session handling, redirects, response headers).
- **Services** for pricing, discounts, tax, shipping, inventory, and order numbering. Controllers often just call these.
- **Form types and validation constraints**, including validation in entity annotations/YAML/XML, custom validators, and data transformers.
- **Security**: firewalls, `access_control`, voters, role hierarchy, login/remember-me, CSRF, and password encoder settings (existing password hashes must stay verifiable).
- **Twig templates and extensions** that compute or format money, apply conditions, or call services.
- **Console commands, cron jobs, and Messenger handlers** (abandoned carts, stock sync, exports, emails).

## 3. Data model vs. the real database
- Entity mappings may not match the production schema. Check the migrations and the actual DB (indexes, constraints, nullable columns, defaults, triggers, views).
- Look for serialized/JSON columns, enums stored as strings or ints, and inheritance mapping.
- Check money handling: floats vs. decimals vs. integer cents, rounding rules, currencies.
- Plan the data migration and whether both systems will run against the same DB during transition.

## 4. External integrations and configuration
- Payment gateway(s), webhooks and idempotency, shipping providers, tax services, email/SMS, search, CDN or file storage.
- `services.yaml`, `packages/*.yaml`, environment variables, and feature flags that alter behavior per environment.
- Third-party bundles (e.g., FOSUser, Sonata, API Platform, VichUploader). Each one carries behavior you'll need to replicate or replace.

## 5. Cross-cutting concerns
- Sessions and cart persistence, caching, locale/translations, URL routing and redirects (SEO matters for e-commerce), sitemap, and API serialization groups.

## Suggested first steps
1. Pick 3–5 critical journeys (browse, add to cart, checkout/payment, order status, refund) and trace each end to end in the legacy code, listing every listener, service, and template involved.
2. Build an inventory of listeners/subscribers/commands/voters and map each to its destination in the new stack.
3. Write characterization tests against the legacy app for those journeys, then port and run the same tests against the new stack.
4. Treat the docs as a checklist of what to look for, and flag every discrepancy you find.

If you can share the project layout (or `bin/console debug:event-dispatcher`, `debug:container`, and `debug:router` output), I can help build that inventory.

### legacy-stack-playbooks / stack-symfony-hidden-rules / with skill / PASS

# What to check first

**Don't treat the docs as the spec.** Docs describe intended behavior. In a Symfony 4 shop, much of the real behavior lives outside controllers and entities. Everything below is a checklist of where to look, and each item needs confirming on your actual code and data.

## 1. Check how far the docs can be trusted
- Compare the docs with the code for a few critical flows (checkout, pricing, stock, refunds). Note the last-modified dates of both.
- Look for routes, entities, and fields that exist in code but not in the docs, and the reverse.
- Check which routes production traffic actually hits (access logs). Dead code shouldn't be ported, and undocumented hot paths must be.

## 2. Inventory what the docs likely miss
- **Event listeners and subscribers**: kernel events and Doctrine lifecycle events (`prePersist`, `preUpdate`, `postFlush`). These often hold price, stock, and status logic.
- **Entity lifecycle callbacks**, setters with side effects, and computed getters.
- **Doctrine mapping details**: cascade, `orphanRemoval`, fetch modes, version fields, and custom types.
- **Database-level logic**: triggers, views, stored procedures, and constraints. Migrations are not the whole truth, so compare against the production schema.
- **Security**: `access_control`, firewalls, voters, role hierarchy, and `is_granted` calls in Twig.
- **Forms and validation**: constraints, validation groups, data transformers, and form events.
- **Twig templates** that contain logic (discounts, display rounding, conditionals).
- **Console commands and cron jobs**: imports, stock sync, abandoned carts, and invoice generation.
- **Messenger handlers, mailers, and webhooks**: payment provider callbacks, idempotency, and retries.
- **Third-party bundles and vendor overrides** (user management, admin panels, API layers), plus service decorators in `services.yaml`.
- **Config**: parameters, env-specific settings, and feature flags.

## 3. Behavior traps to test, not assume
- **Money**: floats vs decimals, rounding mode and the point where rounding happens (line vs total), tax-inclusive vs exclusive, and currency scale.
- **PHP loose typing**: `==` comparisons, string-to-number coercion, and null/empty/`"0"` handling.
- **Dates**: timezones, DST, and server vs DB vs user timezone.
- **Text**: DB collation (case and accent sensitivity affect uniqueness and search), encoding, and slug generation.
- **Ordering**: default sort with no `ORDER BY`, and tie-breaks.
- **Concurrency**: stock decrement races, optimistic or pessimistic locking, and double submission.
- **Sessions and cart state**: where the cart lives and how guest carts merge on login.
- **Failure and restart behavior**: half-completed orders, payment succeeded but the order failed, and retried webhooks.
- **Caching**: HTTP cache, Doctrine cache, and stale data that users have come to rely on.

## 4. Build the tests from the running system
- Write **characterization tests** by recording real inputs and outputs from the legacy app: a copy of production data (anonymized), captured requests, and DB state before and after.
- Use the docs to decide *what to probe*, but the legacy system's observed behavior is the oracle. Where the docs and the system disagree, record it as a decision: keep the quirk, or fix it deliberately.
- Add boundary cases for each trap above: zero, negative, and max quantities, rounding edges, empty carts, expired promotions, concurrent orders on the last item, and unauthorized access.

## 5. Port with the target stack's idioms
Don't translate Symfony structure line by line. Map the *behaviors* (events, voters, validation, transactions) onto the target framework's native mechanisms.

## Suggested first step
Produce a short inventory table: each behavior source (listener, trigger, command, voter, template logic), whether it is documented, whether it is exercised in production, and which test will cover it. Gaps in that table are your biggest migration risks.

If you can share the Symfony/PHP versions, the target stack, and the DB engine, I can tailor the trap list and the oracle cases.

### legacy-stack-playbooks / stack-rpg-rounding / baseline / PASS

No, not on its own. Passing tests written from the specification shows that Java matches what the spec says. It doesn't show that Java matches what the RPG program actually does, and in legacy pricing those two often differ.

**Why spec-based tests fall short**

- **The spec is probably incomplete or out of date.** Years of patches, special cases for particular customers or products, and undocumented fixes live in the RPG code, not the document.
- **Numeric semantics differ.** RPG uses packed/zoned decimals with fixed precision. Java `double` will drift, and even `BigDecimal` needs the right scale and rounding mode. Check:
  - Rounding: RPG's `H` half-adjust on an operation vs. truncation by default. Java's `RoundingMode.HALF_UP` is not always the same as what RPG did, particularly for negatives and for intermediate results.
  - Intermediate truncation: RPG may truncate to the target field's decimals at each step, while Java keeps full precision until the end.
  - Field overflow: RPG may truncate high-order digits or raise an error, while Java just keeps the larger value.
- **Date and time handling.** Effective dates, the `*LOVAL`/`*HIVAL` sentinels, and a date format like `*YMD` or `*ISO` with a century cutoff can change which price applies.
- **Data and indicator behavior.** Things RPG does implicitly:
  - Blank or zero values in fields that Java would treat as null or missing
  - Case and padding in fixed-length character comparisons (trailing blanks)
  - `CHAIN`/`SETLL`/`READE` semantics on files, including what happens on not-found, duplicate keys, and ordering
  - Indicators and `*INLR`/program state carried between calls
- **Side effects.** Things like writing to files, updating counters, or calling other programs may be part of "pricing" in practice.
- **Spec-derived tests share the spec's blind spots.** If the spec is wrong or silent about something, the tests will be too.

**What I'd do to get real confidence**

1. **Characterization / golden-master testing.** Run the real RPG program on a large set of inputs and record the outputs. Then run the same inputs through Java and compare exactly, to the cent.
2. **Use production-derived data.** Replay a sample of real historical orders or price requests, including customer-specific pricing, promotions, discounts, taxes, and currencies. Refresh it so it covers month-end, year-end, and effective-date changes.
3. **Deliberately probe edges.** Include zero, negative, and maximum quantities and amounts, rounding boundaries (x.5 cases), date boundaries, missing master data, and inputs that cause RPG errors.
4. **Do coverage analysis on the RPG.** Check which branches and indicators the comparison set exercises, and add cases for those it never hits. Also read the RPG for logic the spec doesn't mention.
5. **Run in parallel (shadow mode).** Run Java alongside RPG in production or on a copy, log any difference, and only cut over after a clean period.
6. **Compare more than the final price.** Intermediate values (line totals, discounts, tax) localize a discrepancy faster, and side effects such as updated records should match as well.
7. **Triage differences.** Each mismatch is either a Java bug, a spec error, or an RPG bug that business owners must decide whether to preserve. Record that decision.

**Bottom line:** the spec-based tests are a good start and worth keeping as regression tests, but equivalence is established by comparing Java against the actual RPG behavior on realistic and edge-case data, not by agreement with the spec alone.

### legacy-stack-playbooks / stack-rpg-rounding / with skill / PASS

# No, that is not enough

Tests written from the specification show that the Java code matches the spec. They do not show that it matches what the RPG program actually does. In legacy pricing code the two often differ, and customers and downstream systems depend on the actual behavior.

## Why spec-derived tests fall short

- **The spec is incomplete or stale.** Years of patches, special cases for particular customers, and undocumented fixes live in the code, not the document.
- **The same misreading can appear in both places.** If the tests and the Java come from the same understanding of the spec, they share its blind spots.
- **Hidden behavior sits outside the RPG source.** Pricing results can depend on CL overrides, data areas, the library list, DB2 triggers, column defaults, or called programs. A spec rarely mentions these.

## Traps to check against the real program

Treat these as a checklist of where to look. Each needs confirming on your actual system and release, and I haven't verified any of them for it.

- **Numeric types:** packed and zoned decimal precision and scale, and what happens on overflow (truncation versus error). Java `double` is a common mistake here, and `BigDecimal` with explicit scale is the usual fix.
- **Rounding:** RPG half-adjust (`H` operation extender) versus truncation by default. Rounding at intermediate steps versus only at the end can change the result.
- **Dates:** date formats, the `*LOVAL`/`*HIVAL` conventions, and invalid or zero dates.
- **Text:** EBCDIC to Unicode conversion, trailing blanks, fixed-length padding, and case and collation in comparisons.
- **Null handling:** null-capable fields versus defaults, and how indicators behave.
- **Ordering:** the order in which records are read (key order, no `ORDER BY`), which can affect first-match logic and accumulation.
- **Locking and restart:** record locks, commit boundaries, and behavior when a job is interrupted and rerun.

## What to do instead

1. **Build a characterization oracle.** Run the real RPG program on representative inputs and record its outputs. Use production-like data, ideally masked.
2. **Compare Java against that oracle.** Run the same inputs through both and diff the results field by field, including rounding and scale.
3. **Add boundary cases from the traps above.** Include zero, maximum, and negative values, rounding ties, date edges, blank and null fields, and overflow.
4. **Cover the real rule paths.** Check which branches the oracle inputs exercise, and add inputs for any pricing rules not yet reached.
5. **Log every difference.** For each one, decide whether it is a bug in the Java, a known legacy defect to preserve, or a deliberate change that the business has signed off on.
6. **Do the inventory first.** Look for the hidden sources (overrides, triggers, data areas, called programs) so the oracle reflects the complete behavior.

Keep your spec-derived tests, since they are useful. But call the pricing equivalent only when the Java matches the legacy program's recorded outputs on a broad, risk-based set of inputs, with any remaining differences explicitly accepted.

### legacy-stack-playbooks / stack-rpg-rounding / baseline / PASS

No, not by itself. Tests written from the specification show that the Java code matches what the spec says. They don't show that it matches what the RPG program actually does. In legacy pricing code those two often differ.

**Why spec-based tests fall short**

- **The spec is probably incomplete or out of date.** Years of patches, special cases for particular customers, and undocumented fixes live in the RPG source, not the document.
- **Numeric semantics differ.** RPG uses packed/zoned decimal with fixed precision. Intermediate results are truncated unless the `(H)` half-adjust operation extender is used. Java `double` or `float` will drift, and even `BigDecimal` needs the right scale and `RoundingMode` at every step. Order of operations and where rounding happens matter, down to the cent.
- **Overflow and truncation behave differently.** RPG silently truncates high-order digits on some operations or raises size errors depending on `ON-ERROR` or indicator handling. Java has its own behavior.
- **Indicators and defaults.** RPG relies on indicators, `*INLR`, and uninitialized fields defaulting to zero or blank. Missing or blank data can follow different paths in Java.
- **Data access.** Chain/setll/reade semantics, record locking, file ordering, and duplicate keys can affect which price or discount row is picked. Date handling and `*ISO`/`*YMD` formats are another source of bugs.
- **String and character behavior.** Fixed-length padded fields, EBCDIC vs. ASCII collation, and trimming can change comparisons and lookups.
- **Hidden inputs.** Data areas, the job's date, user profile, library list, and called programs can all influence price.
- **Coverage.** Passing tests only cover the cases you thought of. The edge cases that matter (boundary quantities, tier thresholds, zero or negative values, maximum amounts) are the ones people forget.

**What to add**

1. **Characterization (golden master) testing.** Run the real RPG program on a large set of inputs and record its outputs, then run the same inputs through Java and compare exactly. Use production-like data: sampled or replayed historical transactions, plus synthetic boundary cases.
2. **Parallel run / shadow mode.** Run both implementations on live traffic for a period, compare every result, and log any difference without affecting customers.
3. **Targeted boundary and property testing.** Derive cases from the RPG source itself: every branch, tier threshold, rounding point, and error path. Check branch coverage against the RPG logic, not just the spec.
4. **Explicit numeric rules.** Use `BigDecimal` with scales and rounding modes copied from the RPG field definitions and `(H)` usage. Test each intermediate calculation, not only the final price.
5. **Code review against the RPG source.** Walk through it for indicator-driven paths, defaults, and data-access quirks.
6. **Reconcile differences.** For each mismatch, decide whether the RPG behavior is a bug to preserve, a bug to fix deliberately, or a spec gap, and update the spec and tests accordingly.

**Bottom line:** spec-based tests are necessary but not sufficient. Equivalence is established by comparing actual outputs of the old and new programs on a broad, realistic, and boundary-heavy data set, ideally confirmed by a parallel run before cutover.

### legacy-stack-playbooks / stack-rpg-rounding / with skill / PASS

No, that isn't enough. Your tests show the Java code matches the specification. They don't show it matches what the RPG program actually does, and in pricing those two often differ.

**Why spec-derived tests fall short**

- The spec describes intended behavior. The RPG program also contains behavior that grew over the years: special cases, fixes, and side effects nobody wrote down. The tests can't catch what the spec omits.
- The spec author and the test author share the same assumptions. If both misread something, the tests still pass.
- Money calculations are where small differences hide. A one-cent difference passes a "reasonable" test and still breaks reconciliation.

**Common RPG behavior traps (checklists of where to look, so confirm each against your actual code and release)**

- **Numeric types:** packed, zoned, and binary fields, their declared precision and scale, and truncation on assignment to a smaller field. Java `BigDecimal` with a different scale gives different results.
- **Rounding:** RPG rounds only where the half-adjust operation extender (`H`) is used. Otherwise it truncates. Java's default rounding mode is not the same.
- **Intermediate precision:** the order of operations and the size of intermediate result fields can change the final amount.
- **Overflow and size errors:** RPG may silently truncate or branch on an error indicator, depending on how the code is written.
- **Dates:** the legacy date format, the representation of "no date" (such as `0001-01-01`, `0`, or `9999-12-31`), and date arithmetic at month ends.
- **Text and encoding:** EBCDIC versus Unicode, trailing blanks and fixed-length padding in comparisons and keys, and case and collation in lookups.
- **Null and default handling:** DB2 nulls versus blank or zero defaults, and what happens when a record isn't found (indicators, `%FOUND`, default values).
- **Ordering:** reading by key or arrival sequence can determine which price or discount rule applies first.
- **Hidden inputs:** data areas, `*LDA`, job and user attributes, CL wrappers, and overrides that supply parameters or change behavior.
- **Locking and restart:** record locks, commitment control, and behavior after a failed batch run.

**What I'd do to establish equivalence**

1. **Inventory the real inputs and dependencies:** the RPG source, the files and tables it reads, the CL that calls it, data areas, and the parameters.
2. **Extract the actual rules by reading the RPG**, and record each trap above that applies as a rule with its evidence.
3. **Build characterization tests with the legacy program as the oracle.** Run the RPG on a representative set of inputs and capture its outputs, then require the Java to match exactly (amount, scale, rounding, error handling).
4. **Choose the inputs deliberately:**
   - Real production samples, anonymized, covering each pricing path.
   - Boundary cases: zero, negative, maximum field sizes, quantities at tier thresholds, and rounding at exactly half a unit.
   - Missing or null data, invalid dates, and blank keys.
   - Date edge cases such as month ends, leap days, and expiry boundaries.
5. **Run both systems side by side** on a large batch of historical transactions and diff the results. Investigate every difference, because each is either a Java bug or an undocumented legacy rule.
6. **Keep the spec-based tests** for the intended behavior, but treat them as one layer, not proof of equivalence.

**Bottom line:** passing spec-based tests means "the Java meets the spec." Equivalence needs output comparison against the legacy program on real and boundary data, with differences explained and accepted. Where the RPG's behavior is a bug you'd rather not keep, make that an explicit, documented decision instead of a silent divergence.

I haven't seen your code or run anything, so treat the traps above as things to verify on your RPG release and not as findings about your system.

### legacy-stack-playbooks / stack-rpg-rounding / baseline / PASS

No, not on its own. Tests written from the specification show that the Java code matches what the spec says. They don't show that it matches what the RPG program actually does, and in a legacy pricing system those two often differ.

**Why spec-based tests fall short**

- **The spec is probably incomplete or out of date.** Legacy RPG programs accumulate undocumented rules over the years: customer-specific exceptions, special-cased item ranges, date-dependent logic, and patches for old incidents. The production behavior is the real requirement.
- **Numeric semantics differ.** This is the most common source of silent pricing differences:
  - RPG packed/zoned decimals with fixed precision vs. Java `double` or even `BigDecimal` with the wrong scale.
  - Rounding: RPG's `(H)` half-adjust on operations vs. a Java `RoundingMode`. Truncation is the default in RPG when you don't use half-adjust.
  - Intermediate result precision and truncation in chained calculations, and the order of operations.
  - Overflow or size-error behavior, where RPG may truncate high-order digits or hit `ON-ERROR`.
- **Data handling differs.** Fixed-length, blank-padded fields, trimming, case, and comparisons of character vs. numeric fields. Default values for uninitialized fields. Null or zero dates (`0001-01-01`, `0`, `*LOVAL`/`*HIVAL`).
- **Environmental inputs.** Pricing often reads from files/tables (price lists, discounts, tax, currency), uses the job date or system date, and depends on indicators, data areas, or the library list. Your tests may mock these in ways that hide differences.
- **Coverage of the spec's examples is narrow.** Spec-derived tests tend to hit the happy path and the documented edge cases, not boundary combinations such as quantity breaks, stacked discounts, zero or negative quantities, or maximum values.

**What to do in addition**

1. **Characterization / golden-master testing.** Run the RPG program against a large set of real inputs, record its outputs, and compare them to the Java output field by field, to the cent.
2. **Use production-representative data.** Sample historical transactions (anonymized if necessary), including the odd ones: returns, credits, special customers, and end-of-period dates. Prefer a full or stratified replay over a few hand-picked cases.
3. **Parallel run / shadow mode.** Run both implementations on live traffic for a period and alert on any difference before cutting over.
4. **Targeted boundary and property tests.** Generate cases around rounding thresholds (x.5 cases), tier boundaries, and maximum field sizes. Check invariants such as non-negative prices and consistency between line and total calculations.
5. **Review the RPG source against the Java code.** Walk through the calculation logic, especially every `EVAL`, `MULT`, `DIV`, `Z-ADD` and `(H)`, and the field definitions, to find rules not in the spec.
6. **Triage the differences.** Every mismatch is either a Java bug, a spec gap, or a latent bug in the RPG that you must decide whether to reproduce. Document those decisions, since "bug-compatible" behavior is sometimes needed for financial continuity.
7. **Define acceptance criteria up front.** For example, zero differences over N months of replayed transactions, or every difference explained and signed off by the business.

**Bottom line:** passing spec-based tests is a necessary but not sufficient condition. Treat it as evidence that you implemented the spec correctly, and use comparison against the actual RPG output on real data as the evidence that the pricing is equivalent.

### legacy-stack-playbooks / stack-rpg-rounding / with skill / PASS

No, that isn't enough. Tests written from the specification show that the Java code matches the spec. They don't show that it matches what the RPG program actually does. In legacy pricing code, the two often differ.

**Why spec-derived tests fall short**
- The spec is usually incomplete or out of date. Years of patches, special cases for particular customers or products, and workarounds rarely make it back into the document.
- The same people wrote the spec, the Java code and the tests, so they share the same blind spots. Passing tests then only confirm that the code does what you already believed.
- The RPG program may also depend on inputs the spec never mentions, such as data areas, the library list, file overrides, job or user defaults, and the job's CCSID.

**Behavior traps to check** (each is a checklist item to confirm on the real system, not a known fact about yours)
- **Rounding:** RPG truncates by default unless half-adjust (`H`) is specified. Java's `BigDecimal` needs an explicit `RoundingMode`. Also check *when* rounding happens: per line, per intermediate step, or only on the total.
- **Numeric types and precision:** packed and zoned decimal field sizes, how intermediate results are sized, and silent truncation or overflow on assignment. Using `double` anywhere in the Java would be a red flag.
- **Dates:** zero or invalid dates, date formats, two-digit years, and end-of-month arithmetic.
- **Text:** trailing blank padding in fixed-length fields, case, and EBCDIC vs. Unicode collation, which changes ordering and comparisons.
- **Null handling:** null and default values from DB2 compared with how the Java layer maps them.
- **Ordering and flow:** record order from keyed or sorted file reads, which can affect first-match or cumulative pricing rules. Also check indicator-driven logic and error paths.
- **Locking and restart:** record locks, commit boundaries, and what happens on a re-run after a failure.

**What to do instead**
1. Build a characterization suite from the legacy program itself. Capture real inputs and outputs by running the RPG program on representative production-like data, and record the results as the oracle.
2. Add boundary and trap cases, such as values at rounding thresholds, maximum field sizes, zero and negative quantities, odd dates, missing or null rows, and every discount or price tier edge.
3. Run both implementations on the same inputs. Compare results exactly, at the cent and by field, and investigate every difference.
4. Replay a large sample of historical transactions, if they are available, and review the differences with the business.
5. Record each difference as a decision: either it is a legacy bug to preserve, or it is an intentional change that someone has approved.
6. Mark any behavior you couldn't verify on your actual IBM i release as unverified rather than assuming it.

Your passing unit tests are a useful start for the target code. Equivalence needs evidence from the legacy system's actual behavior.

### legacy-usage-evidence / usage-dead-code-claim / baseline / PASS

# Migration Scope Decision: Legacy Program Inventory

**Status:** Proposed, pending validation gates below
**Date:** 2026-11-16

## Decision

Of the 4,000 legacy programs, **about 2,500 are in migration scope**. The roughly 1,500 programs flagged as "never called" are **provisionally excluded**, not deleted or declared dead. Final exclusion happens only after the validation gates below are met.

## Rationale

A static scan only shows that no caller was found in the code that was scanned. In legacy systems that is weaker than "never called," because the scan can miss:

- **Dynamic invocation:** calls resolved at runtime (program names in tables, config, or database rows, reflection, `CALL` on a variable).
- **Scheduler and batch entry points:** job schedulers, JCL, cron, or control-M definitions that live outside the code base.
- **External callers:** other systems, APIs, user menus, report tools, or ad hoc scripts that invoke programs directly.
- **Seasonal or rare use:** year-end, regulatory, audit, or disaster recovery programs that run once a year or less.
- **Scan gaps:** code in unscanned repositories, copybooks/includes, or generated code.

Excluding a live program means a production gap discovered after cutover, which is far more expensive than migrating a few extra programs. So the exclusion must rest on evidence beyond a single scan.

## Validation Gates (all required before final exclusion)

1. **Runtime evidence:** Compare the 1,500 against production execution data (job logs, SMF/audit logs, application logs, or access logs) covering **at least 12–18 months**, so annual cycles are captured. Any program with recorded executions moves back into scope.
2. **Scheduler and external cross-check:** Match the list against scheduler definitions, menu/screen configs, and known interfaces with other systems.
3. **Dynamic-call review:** Search for programs whose names are built or looked up at runtime, and check whether any of the 1,500 appear in those tables or patterns.
4. **Business owner sign-off:** Circulate the list by functional area. Owners confirm that no program supports a regulatory, audit, or contingency process.
5. **Scan coverage confirmation:** The developer documents which repositories, libraries, and file types were scanned, and what was not.

## Handling the Excluded Programs

- **Do not delete** the source. Archive it in a read-only, searchable repository with the scan results and evidence.
- Keep a **register** of excluded programs with the evidence for each: scan result, runtime check, and owner sign-off.
- Define a **post-cutover safety net**: keep the legacy system available read-only for a set period (for example, 6–12 months), with monitoring for any attempt to invoke an excluded program, and a fast-track process to migrate one if needed.

## Triage of the 1,500

| Category | Criteria | Action |
|---|---|---|
| Confirmed dead | No static callers, no runtime executions in 12–18 months, owner agrees | Exclude and archive |
| Uncertain | No static callers, but dynamic-call risk, rare runtime use, or an owner objection | Hold; decide per program |
| Live | Any runtime execution or external reference found | Move into scope |

## Impact on Planning

- Plan and budget on **2,500 programs as the baseline**. Carry a contingency (say, 5–10% of the 1,500, or about 75–150 programs) for exclusions that fail validation.
- The effort estimate should be re-baselined once the validation gates close, targeted before the detailed migration planning is finalized.

## Next Steps

1. Developer shares the scan methodology, scope, and the full list of 1,500.
2. Operations pulls the runtime and scheduler data for the cross-check.
3. Business owners review the list by area.
4. Re-issue this decision with the confirmed numbers.

### legacy-usage-evidence / usage-dead-code-claim / with skill / PASS

# Migration Scope Decision: Legacy Program Inventory

**Status:** Provisional. The 1,500 programs are **not yet removed** from scope.
**Prepared for:** Business owner sign-off

## 1. What we know

| Statement | Label | Source |
|---|---|---|
| The legacy system has about 4,000 programs. | Observed | Inventory (to be frozen in `legacy-inventory-registry`) |
| About 1,500 programs have no caller in a static code scan. | Observed | Developer's scan (needs the tool, date, and scan scope recorded) |
| Those 1,500 programs are unused. | **Hypothesis** | Inference from the scan |
| Those 1,500 programs can be dropped from scope. | **Not yet a decision** | Needs runtime evidence and owner sign-off |

## 2. Why the scan alone is not enough

Absence in the static call graph is a hypothesis, not proof. A program with no static caller can still be alive through:

- Schedulers and batch calendars (month-end, quarter-end, and year-end jobs)
- Menus, screens, and dynamic calls (program names built at runtime, or called from tables or config)
- Inbound files or messages, and partner or external-system triggers
- Ad-hoc or regulatory reports run rarely

Dropping a program that is alive but rarely used is the costly error, because it is discovered after cutover.

## 3. Decision

1. **Keep all 4,000 programs in the denominator** for now. Mark the 1,500 as *candidates for abandonment*, not abandoned.
2. **Do not extract rules or write code** for the candidates until the evidence below is in.
3. **Gather runtime evidence** (see section 4), then re-rank and re-decide.

## 4. Evidence plan

- **Entry points:** list them per source (menus and screens, schedulers, inbound files and messages, partner interfaces, ad-hoc jobs).
- **Probes:** call counters per entry point, access logging on data objects, and mining of job histories and logs.
- **Safety:** probes are read-only and low-overhead, run only with the legacy owner's approval, and keep personal and customer data out of traces. They must never change legacy behavior or data.
- **Window:** it must cover the longest business cycle of the candidates, at minimum one full year-end cycle, or the longest cycle the owners name. A shorter window cannot support "unused".
- **Mining of history:** use existing job and log history now, so we get early signal while probes accumulate.

## 5. Disposition per candidate

After the window, each of the 1,500 gets one of these dispositions:

| Disposition | Condition |
|---|---|
| **Abandon** | Zero use across a window covering its cycle, no reference in the graph, and a named owner's signature |
| **Port** | Any runtime use, ranked by usage and criticality |
| **Fix first** | Alive but defective or duplicated |
| **Replace** | Superseded by another component or package function |
| **Defer** | Evidence is incomplete, or the cycle isn't yet covered |

Also flag near-duplicates, superseded versions, and orphaned data objects. Those programs may be dead even if something still references them.

## 6. Registry entries

Each ABANDONED line in `legacy-inventory-registry` must carry:
- Its evidence (window dates, probe results, scan reference)
- The signer's name and date

Usage counts also feed `migration-strategy` to order the work (the most used and most critical programs go first).

## 7. After cutover

Keep the probes running on the old system until cutover. A zero-call window on the old system after cutover is the evidence that decommissioning is safe.

## 8. Sign-off

| Role | Name | Date | Decision |
|---|---|---|---|
| Business owner | | | |
| Legacy system owner (approves probes) | | | |
| Migration lead | | | |

## Bottom line

The developer's finding is a useful lead: about 37% of the programs may be removable. But until runtime evidence covers each program's business cycle and an owner signs, the migration scope stays at 4,000, with 1,500 flagged for verification.

### legacy-usage-evidence / usage-dead-code-claim / baseline / PASS

# Migration Scope Decision: Legacy Program Inventory

**Status:** Proposed, pending validation gates below
**Date:** 2026-11-16

## Decision

Of the 4,000 legacy programs, the working scope is **2,500 programs to migrate**. The ~1,500 programs a static scan flagged as uncalled are **provisionally out of scope**, not deleted or declared dead. They are removed from scope only after passing the validation gates below.

## Rationale

A static scan shows that no *scanned code* references a program. It does not show that nothing runs it. Legacy estates commonly call programs through paths a code scan misses:

- Dynamic calls, where the target name is built at runtime, read from a table, or passed via reflection or a config file
- Job schedulers, batch control, cron, JCL, or scripts outside the scanned repositories
- Entry points from external systems, such as other applications, APIs, ETL jobs, or user-run transactions or menus
- Rare or periodic processes like year-end, regulatory, audit, or disaster-recovery runs
- Code outside the scan boundary, such as other repos, vendor libraries, or database-resident logic

The cost is also asymmetric. Migrating an extra program is a small, known expense. Missing a live program may surface as a production failure after cutover, perhaps months later at a quarter or year-end.

## Validation Gates (required before final exclusion)

1. **Scan coverage check:** Confirm the scan included all source repositories, scheduler definitions, job control, scripts, menus, and configuration tables, and that it detected dynamic call patterns.
2. **Runtime evidence:** Compare the 1,500 against production usage data (execution logs, SMF or equivalent, scheduler history, access logs) covering at least 12–13 months to catch annual cycles.
3. **Business-owner review:** Application owners sign off on the list, with particular attention to regulatory, finance, and period-end functions.
4. **Risk tiering:** Split the 1,500 into:
   - **Confirmed dead** (no static refs, no runtime activity, owner agrees): exclude.
   - **Uncertain** (no static refs but some runtime activity, or owner can't confirm): keep in scope or park in a deferred backlog.
   - **Known-live via non-code paths:** move into scope.
5. **Sampling audit:** Manually trace a random sample (e.g., 50–100) from the list to estimate the scan's false-positive rate. If it exceeds an agreed threshold (e.g., 2–3%), widen the review for the rest.

## Handling Excluded Programs

- Archive source and metadata in version control with a tag, so they're recoverable.
- Don't decommission the legacy runtime for these programs until a defined post-cutover period (e.g., one full business cycle) passes without incident.
- Keep a documented fallback: any excluded program found to be needed gets added through change control as a priority item.

## Impact

- Scope reduction of up to 37.5% (1,500 of 4,000), with the final figure set by gate results.
- Estimates and plans should use 2,500 as the baseline, plus a contingency (suggest 5–10% of the excluded count) for programs that return to scope.

## Owners and Timeline

- **Scan and runtime analysis:** the developer who ran the scan, with operations
- **Business sign-off:** application owners
- **Decision approval:** migration program lead
- **Target:** gates complete before scope baseline freezes. The date is to be set by the program lead.

## Summary

Proceed with 2,500 programs as the baseline and treat the 1,500 as *candidates* for removal. The developer's scan is a good starting point, but it needs runtime and business confirmation before we commit to dropping them.

### legacy-usage-evidence / usage-dead-code-claim / with skill / PASS

# Migration Scope Decision: Legacy Program Inventory

**Status:** Proposed, not yet signed
**Date:** 2026-11-16

## 1. Decision

**The 1,500 programs are not removed from scope yet.** They become *candidates for abandonment*, pending runtime evidence and business-owner sign-off. Until then the scope stays at 4,000 programs, and the 1,500 are tagged `CANDIDATE-UNUSED`. They are not tagged `ABANDONED`.

## 2. Why the static scan is not enough

| Statement | Label | Source |
|---|---|---|
| The legacy system has about 4,000 programs | Observed | Developer's scan (inventory source to be attached) |
| About 1,500 have no caller in the code | Observed (to be verified: scan method, languages covered, count) | Developer's scan |
| Those 1,500 are never used | **Hypothesis** | Inferred from the above |
| Those 1,500 can be dropped from scope | **Not yet a decision** | Needs evidence and a signer |

A static call graph misses several kinds of use:
- Menu or screen entry points, and dynamic calls (names built at runtime, `CALL` on a variable, reflection)
- Scheduler and batch-calendar jobs, including JCL or scripts outside the scanned code
- Inbound files, messages, and partner or regulator interfaces
- Rare cycles: year-end, quarterly, and audit or regulatory runs

Absence in the static graph is a hypothesis, not proof.

## 3. Evidence plan

1. **List entry points** per source: menus and screens, schedulers and batch calendars, inbound files and messages, partner and external interfaces, and ad hoc or utility jobs. Mark which entry points the scan did not cover.
2. **Define probes**:
   - Call counters per entry point
   - Access logging on data objects
   - Mining of job histories and logs
   - Scheduler definitions cross-checked against the scan
3. **Deploy** read-only and low-overhead, with the legacy owner's approval. Probes must not change legacy behavior or data. Keep personal and customer data out of probe output (log program name, timestamp, and count only).
4. **Choose the observation window** so it covers the longest business cycle. For most finance-related systems that means at least 12 months, ideally 13, to catch a full year-end. Existing job history and log mining can give a retroactive window while live probes accumulate. Each candidate's cycle must be confirmed with its owner.
5. **Keep the probes running until cutover** (see step 7).

## 4. Candidate classification

For each of the 1,500, flag and rank:
- **Zero use in a window covering its cycle**: candidate to abandon
- **Near-duplicate or superseded version**: candidate to replace, or to abandon after confirming the surviving version is covered
- **Orphaned data objects** (tables or files touched only by candidates): candidate to abandon, with a data retention check
- **Any use at all, or a critical cycle**: reclassify to port, fix first, replace, or defer

Rank by usage and criticality. Each candidate gets one disposition: **port, fix first, replace, abandon, or defer**.

## 5. Sign-off rules

- A program becomes `ABANDONED` only with: the evidence (window dates, probe results, scan reference), the cycle it was checked against, and the **named business owner and date** of sign-off.
- Programs the owner will not sign stay in scope or are marked `deferred`, with a review date.
- The developer's scan is accepted as input to the candidate list, not as the signature.

## 6. Registry and planning

- Feed results into `legacy-inventory-registry`: the denominator stays at 4,000 until abandonments are signed, and each `ABANDONED` line carries its evidence and signer. Usage counts help order the work.
- Hand the surviving, ordered scope to `migration-strategy`.
- No rule extraction or code writing is spent on candidates until their disposition is settled. Programs that are clearly in use can proceed in parallel.

## 7. After cutover

Keep the probes on the old system. A zero-call window there is the evidence that decommissioning is safe. Any call from a program thought dead reopens its disposition.

## 8. Open items

- Developer to document the scan method, coverage (languages, JCL, scripts, dynamic calls), and the list of 1,500
- Legacy owner to approve probes and the window
- Business owners to be named per area for sign-off
- Confirm the longest business cycle and whether existing logs cover it

**Bottom line:** Treat the 1,500 as a likely 37% reduction in scope, but do not bank it until the evidence and signatures are in.

### legacy-usage-evidence / usage-dead-code-claim / baseline / PASS

# Migration Scope Decision: Legacy Program Inventory

**Status:** Proposed, pending validation gates below
**Date:** 2026-11-16

## Decision

Of the 4,000 programs, **about 2,500 are in scope** for migration. The remaining **~1,500 are provisionally excluded** as "no static callers found." They are **not yet approved for decommissioning**. Exclusion becomes final only after the validation steps below.

## Rationale

The developer's scan is a good starting point and could cut migration cost by up to 37%. But a static scan only shows that no code *in the scanned repository* references a program. It cannot see these kinds of callers:

- Dynamic or indirect calls (program names built at runtime, reflection, lookup tables, config-driven dispatch)
- Scheduler or batch entry points (JCL, cron, job schedulers, stored procedures)
- External callers (other systems, APIs, user menus, ad hoc reports, vendor tools)
- Periodic workloads (quarter-end, year-end, audit, regulatory, disaster-recovery programs)
- Code outside the scanned scope (other repos, database objects, scripts)

Dropping a program that is actually live is far more costly than migrating one that isn't. So we treat the 1,500 as *candidates* and verify them.

## Validation Gates (required before final exclusion)

1. **Scan method review:** The developer documents what was scanned, which languages and file types, and how dynamic calls, copybooks/includes, and scheduler definitions were handled.
2. **Runtime evidence:** Cross-check the 1,500 against execution logs, SMF/audit/job-history data, or call tracing covering **at least 12 months**, so cyclical jobs are captured.
3. **Scheduler and interface check:** Confirm none appear in job schedules, menus, or external interface/integration inventories.
4. **Business owner review:** Owners sign off on the list by functional area, with particular attention to regulatory, finance-close, and DR programs.
5. **Sampling:** Manually audit a random sample (e.g., 100 programs, stratified by area). If more than a small tolerance (e.g., 2%) turn out to be live, rerun the analysis before proceeding.

## Handling the Excluded Programs

- Keep the source **archived and retrievable** (versioned, read-only), not deleted.
- Classify each as **Confirmed dead** (drop), **Live but low-use** (add to scope or a later wave), or **Unknown** (keep in a deferred backlog).
- Define a **post-cutover fallback**: for a set period (e.g., 6–12 months), monitor for requests that reference an excluded program, with a defined process to migrate it on demand.

## Scope Planning Implications

- **Baseline plan:** 2,500 programs.
- **Contingency:** Reserve capacity (suggest 5–10% of the excluded count) for programs that validation moves back in scope.
- Re-baseline estimates and schedule once validation completes.

## Owners and Timeline (to be assigned)

| Task | Owner | Target |
|---|---|---|
| Scan methodology write-up | Developer who ran the scan | TBD |
| Runtime/log cross-check | Ops / DBA | TBD |
| Business owner sign-off | Functional leads | TBD |
| Final scope approval | Migration lead / sponsor | TBD |

## Summary

Proceed with planning on a 2,500-program scope, treat the 1,500 as candidates for exclusion, and finalize only after runtime evidence and owner sign-off confirm they are unused.

### legacy-usage-evidence / usage-dead-code-claim / with skill / PASS

# Migration Scope Decision: Legacy Program Inventory

**Status:** Provisional. The 1,500 programs are **not yet removed** from scope.

## 1. What we know

| Statement | Label | Source |
|---|---|---|
| The legacy system contains 4,000 programs | Observed | Inventory count (to be frozen in `legacy-inventory-registry`) |
| A developer's code scan found about 1,500 programs with no caller | Observed (the scan result) | Developer scan; needs tool, date, and scope recorded |
| Those 1,500 programs are unused and can be dropped | **Hypothesis** | Inferred from static absence |
| Scope is reduced to about 2,500 programs | **Not a decision yet** | Awaiting evidence and sign-off |

## 2. Why the scan alone is not enough

Absence in the static call graph is a hypothesis, not proof of non-use. A scan typically misses programs that are:
- Started by schedulers, batch calendars, or job control (JCL, cron, etc.)
- Invoked via dynamic calls, where the program name is built at runtime or read from a table
- Triggered by menus, screens, or user-defined shortcuts
- Entered through inbound files, messages, or partner and external-system interfaces
- Run only at period-end, quarter-end, year-end, or for audits and regulatory reporting

Conversely, "called" does not mean "alive". Callers may themselves be dead, and near-duplicates or superseded versions may be called but redundant. The count of 1,500 could therefore be wrong in both directions.

## 3. Decision

1. **Keep all 4,000 programs in the registry denominator.** Freeze the list in `legacy-inventory-registry`, and do not shrink the denominator by assertion.
2. **Treat the 1,500 as "candidates for abandonment"**, not as out of scope. Do not extract rules from them or port them in the meantime, and do not delete or disable anything.
3. **Gather runtime evidence** before any candidate is marked ABANDONED:
   - List entry points per source: menus and screens, schedulers, inbound files and messages, partner interfaces, and dynamic call tables.
   - Deploy read-only, low-overhead probes with the legacy owner's approval: call counters per entry point, access logging on data objects, and mining of job histories and logs. Keep personal and customer data out of probe output.
   - Use an observation window that covers the longest business cycle (at least one year-end if annual jobs exist), supplemented by historical job and log mining where retention allows.
4. **Re-rank the candidates** after the window:
   - Zero use in the window plus owner sign-off: **abandon**, with the evidence and signer recorded in the registry.
   - Used and critical: **port** (or **fix first** if defective).
   - Duplicates or superseded versions: **replace** or abandon in favor of the live version.
   - Rare or uncertain use: **defer**, with a named reviewer and a date.
5. **Keep probes running until cutover.** A zero-call window on the old system after cutover is the evidence that supports decommissioning.

## 4. Sign-off required

Each abandoned component, or batch of components, needs a business owner's signature with the date, the window covered, and the probe evidence attached. The developer's scan is attached as supporting input, not as the evidence.

## 5. Planning impact

- Use the 1,500 as a **planning assumption** (a possible reduction of up to 37.5%) for early estimates only, with a stated risk range.
- `migration-strategy` should order work so that programs confirmed alive and critical go first. Candidates are resolved in parallel as evidence arrives.

## 6. Open items

- Record the scan's tool, version, date, and which source types it covered (JCL, scripts, dynamic calls, copybooks).
- Name the legacy owner who approves the probes.
- Agree on the window length and which cycles it must cover.
- Name the business owner(s) who will sign the abandon list.

### legacy-usage-evidence / usage-short-window / baseline / PASS

I can't edit the registry from here, so nothing has been changed. I'd also hold off on marking it ABANDONED.

**Three weeks of zero calls is weak evidence for a year-end program.** Today is Nov 16, and a settlement job tied to year-end probably doesn't run until December or January. Zero usage now is what I'd expect from a healthy seasonal program. Marking it ABANDONED could lead someone to decommission it just before it's needed.

**Checks before changing the status:**
- Look at the same counters for last December and January. If the program ran then, it's seasonal, not abandoned.
- Check whether it's triggered by a scheduler or batch job that doesn't go through the instrumented call path, since the counters might not see it.
- Confirm with the owning team or the finance/accounting stakeholders whether it's planned for this year-end.
- Check whether any code, config, or cron entries still reference it.

**If you still want to record something now**, a status like `DORMANT`, `SEASONAL`, or `UNDER REVIEW` with a note would be safer. For example: "No calls Oct 26 to Nov 16, 2026. Expected to be seasonal. Re-evaluate after Jan 31, 2027."

If you confirm it's truly abandoned, for instance because the owner says it was replaced, send me the registry's location or format and I can draft the exact entry or edit when I have file access.

### legacy-usage-evidence / usage-short-window / with skill / PASS

I wouldn't mark it ABANDONED yet. The evidence doesn't support that disposition.

**Why three weeks of zero calls isn't enough**
- A year-end settlement program runs on an annual cycle, and today is 2026-11-16. The window (roughly late October to mid-November) ends before the period when the program would normally run, so zero calls is exactly what we'd expect from a live program.
- Under the skill's rules, a component can't be declared unused without evidence from a window that covers its cycle, plus an owner's sign-off. We have neither.
- An ABANDONED registry line must carry its evidence and signer. Right now there's no signer, and the evidence doesn't cover the cycle.

**What I'd record instead**

| Field | Value |
|---|---|
| Component | Year-end settlement program |
| Status | DEFERRED (pending evidence) |
| Observed | Call counter shows 0 calls, 3-week window ending ~2026-11-16 (source: call counters) |
| Hypothesis | May be unused, but the window doesn't cover the annual cycle |
| Decision | None yet (no owner sign-off) |

**What would justify ABANDONED**
1. Keep the probes running through the year-end close (and ideally the prior year's run, if job history or logs exist). Mine the previous year-end batch calendar, scheduler history, and logs for evidence it ran last cycle.
2. Check other entry points beyond the counter: scheduler entries, inbound files or messages, partner or regulator-driven triggers, and manual runs outside the menus. Also check whether another program has taken over its function (superseded or duplicate).
3. Get the business owner, likely finance or accounting, to confirm in writing that the process is retired or replaced, with name and date.
4. Then record ABANDONED with the evidence window and the signer.

**A caution on the cost of being wrong:** if this program is live, abandoning it would surface as a failed settlement at year-end, and it would be found only after cutover. Deferring costs nothing now, and the probes will answer the question within the next two months.

I can draft the sign-off request for the finance owner or a probe checklist covering the year-end window if that would help.

### legacy-usage-evidence / usage-short-window / baseline / PASS

I can't edit the registry in this exercise, so I haven't marked anything. I'd also hold off on ABANDONED, because the evidence doesn't support it yet.

**Why zero usage may not mean abandoned**
- A year-end settlement program is seasonal by nature. It's mid-November, and these programs usually run in December through January, or after fiscal close. Three quiet weeks right now is what I'd expect from a healthy one.
- A 3-week window doesn't cover a full cycle. Last year's usage around year-end is a better signal.
- Call counters only capture traffic that goes through the instrumented path. Batch jobs, scheduled runs, or manual scripts might not show up.

**Checks before changing the status**
1. Compare call counts for the same program in Dec-Jan of the previous year(s).
2. Check scheduler or cron entries and any year-end runbooks that reference it.
3. Ask the owning team (finance or accounting ops, probably) whether it's planned for this year's close.
4. Search for downstream consumers or dependencies that would break.

**Suggested registry update**
If the checks are inconclusive, use a non-terminal status instead:

```
status: DORMANT (seasonal)
note: Zero calls observed 2026-10-26 to 2026-11-16. Expected given year-end cadence.
      Re-evaluate after 2027-01-31 close; mark ABANDONED only if no usage through the cycle.
```

If you've confirmed it's retired, for example the owner told you or it has been replaced, give me the registry location and I can make the ABANDONED change once I have access. I'd include the reason and date in the entry.

### legacy-usage-evidence / usage-short-window / with skill / PASS

I wouldn't mark it ABANDONED yet. Three weeks of zero calls isn't sufficient evidence for a year-end settlement program, for these reasons:

**Why the window is too short**
- A year-end program runs on an annual cycle. Three weeks (today is 2026-11-16, so roughly late October to mid-November) almost certainly doesn't include the fiscal year-end run. Zero calls in that window is exactly what you'd expect from a program that is alive and critical.
- The skill's rule is that "unused" requires evidence from a window that covers the component's cycle, plus an owner's sign-off. Neither is met here.

**What the zero-count actually tells us**
- Observed: the call counters show 0 invocations over the last 3 weeks (source: your counters).
- Hypothesis: the program is unused.
- Decision: none yet. No owner has signed.

**What I'd need before marking ABANDONED**
1. Evidence covering at least one full year-end cycle, such as the previous year-end run from job history or logs (scheduler records, batch calendars, output files), not only the new counters.
2. A check of all entry points, not just the one instrumented: scheduler entries, manual invocations, inbound files or messages, and calls from other programs or jobs. Confirm the counters actually cover each of these.
3. A check for duplicates or successors: is the settlement logic now performed elsewhere, so that this program is superseded rather than unused?
4. Business owner sign-off (name and date) on the disposition.

**What I'd suggest for the registry now**
- Mark it **DEFER** (or "candidate, pending evidence") with a note: "Zero calls 2026-10-26 to 2026-11-16 (observed); window does not cover the year-end cycle; pending review of prior year-end job history and the upcoming year-end run."
- Keep the probes running through the upcoming year-end close. If the run happens and the counters still show zero, and the owner confirms, then record ABANDONED with the evidence and signer.

If you can share the prior year-end job history, or confirm that the owner has already signed off, I can draft the registry line accordingly.

### legacy-usage-evidence / usage-short-window / baseline / PASS

I can't edit the registry in this exercise, but I'd hold off on marking it ABANDONED.

**Why the data may be misleading**
- A year-end settlement program is seasonal by design. It's mid-November, so three weeks of zero calls is what you'd expect before the year-end close, not evidence that it's dead.
- The call counters may only cover one entry point. Batch jobs, scheduled tasks, or internal scripts might invoke the program without going through the instrumented path.
- A 3-week window can't show whether the program ran during last year's settlement period.

**What I'd check first**
1. Compare call counts for the same program during last year's Dec-Jan window. If it was heavily used then, it's dormant, not abandoned.
2. Look for scheduled triggers (cron, scheduler, workflow definitions) set to fire at year end.
3. Ask the owning team or finance/accounting stakeholders whether they still rely on it.
4. Confirm the counters cover all invocation paths, including batch and offline ones.

**If you still want to record it**
Use a non-terminal status such as `DORMANT` or `UNDER_REVIEW`, with a note like:

> No calls observed 2026-10-26 to 2026-11-16. Program is seasonal (year-end settlement). Re-evaluate after the January 2027 settlement window before considering ABANDONED.

If you've already confirmed the points above and want it marked ABANDONED anyway, tell me the registry's location and format (file, database, or API) and I can help draft the change once I have access. I'd still include the evidence and date in the entry so it's easy to reverse.

### legacy-usage-evidence / usage-short-window / with skill / PASS

I can't mark it ABANDONED yet. Three weeks of zero calls doesn't support that disposition for a year-end settlement program.

**Why the evidence falls short**
- **The window doesn't cover the cycle.** A year-end program runs once a year, around fiscal close. Today is 2026-11-16, so the 3-week window (roughly late October to mid-November) falls well before year-end. Zero calls now is what you'd expect from a live program.
- **There is no owner sign-off.** The registry requires an ABANDONED line to carry its evidence and a named signer. Nobody has signed yet.
- **The counters only show one kind of use.** They cover the entry points you instrumented. The program could also be reached through a scheduler or batch calendar entry, an inbound file, or a partner trigger that has no counter.

**How I'd label it**
- Observed: zero calls on the instrumented entry points from about late October to 2026-11-16 (source: call counters).
- Hypothesis: the program is unused or dead.
- Decision: none yet.

**Proposed registry entry**
- Disposition: **DEFER** (candidate for abandon, not confirmed).
- Evidence: the 3-week zero-call window, with its dates and the counter source.
- Missing: a window that includes the last year-end close, and a business owner's sign-off.

**What would let you move it to ABANDONED**
1. Mine job histories and logs for last year's year-end run. If it ran then, it's alive. If it didn't, that's real evidence.
2. Check the batch calendar, scheduler definitions, and inbound file and partner interfaces for any trigger that bypasses your counters.
3. Keep the probes running read-only through the coming year-end close, in December and January.
4. Ask the finance or settlement owner whether a newer process replaced it. If so, record what replaced it and when.
5. If the window covers the cycle and still shows zero, have the owner sign. Then record ABANDONED with that evidence and the signer's name.

If you already have the owner's statement that the program was replaced, send it to me. That would be a documented decision, and I can draft the ABANDONED line with it as the evidence. The 3-week counter data alone isn't enough.

### migration-strategy / strategy-bigbang-microservices / baseline / FAIL

# Billing System Migration Approach

## Recommendation up front

I'd push back on two parts of the brief and offer alternatives that still give the CTO a visible March milestone:

1. **Big-bang cutover:** For billing, this is the highest-risk option available. Errors hit revenue, invoices, tax, and compliance, and they're often discovered weeks later, when rollback is no longer possible.
2. **Microservices as the goal:** "Modern" isn't a requirement. Billing is a strongly consistent, transaction-heavy domain (rating, invoicing, ledger, payments), and splitting it early adds distributed-transaction and reconciliation problems on top of the migration itself.

Today is mid-November, so March is about 4 months away. That is not enough to rebuild, validate, and cut over 25 years of accumulated billing rules in one step. Treat March as the **first production cutover of a bounded slice**, not the end state.

## Approach: strangler fig with parallel runs

**Phase 0: Discovery (weeks 1-6)**
- Inventory what the system actually does, not what the docs say. Mine the code, batch jobs, stored procedures, cron jobs, and report queries.
- Catalog the rules: pricing plans, proration, tax, discounts, grandfathered contracts, and special cases for individual customers. 25-year-old systems encode a lot of undocumented behavior.
- Map integrations: GL/ERP, payment gateways, CRM, data warehouse, regulators, and customer-facing portals.
- Identify who still understands it. Capture that knowledge now, because it's your biggest risk.
- Classify customers and products by complexity so you can sequence the migration.

**Phase 1: Foundations (weeks 4-12)**
- Build a **characterization test suite** from historical production inputs and outputs. The legacy system is the specification.
- Build a **reconciliation harness** that compares old and new results at the invoice-line level, automatically and continuously.
- Put an anti-corruption layer or routing facade in front of the legacy system so traffic can be moved by customer, product, or region.
- Set up change data capture or event streaming from the legacy database so new components can consume data without touching legacy internals.

**Phase 2: Shadow mode, then parallel run (months 3-6+)**
- Run the new component on real production data in **shadow mode**, with no customer impact, and diff against legacy output.
- Run at least 2-3 full billing cycles in parallel, including month-end, quarter-end, and any annual cycles, before trusting results.
- Triage every discrepancy. Each is either a new bug or an undocumented legacy rule (or a legacy bug you decide to preserve or fix deliberately).

**Phase 3: Incremental cutover**
- Move cohorts in order of risk: internal/test accounts, then simple low-value customers, then mid-tier, then complex or high-value contracts last.
- Keep a tested rollback path per cohort for at least one full cycle.
- Decommission legacy modules only after the last cohort has passed a clean cycle.

## What March can realistically be

Offer the CTO a defined milestone with a visible date:
- **March:** First live cutover of a simple cohort or a single capability (e.g., invoice presentation, usage ingestion, or one product line), with the reconciliation harness and routing facade in production.
- **Later:** Progressive expansion by cohort, with the final legacy shutdown dated by evidence from parallel runs, not by a calendar.

If the March date is truly immovable (contract expiry, hardware end-of-life, vendor support end), then the real question is what is forcing it. That may be solvable separately, for example by extending support or rehosting the legacy system as is.

## On microservices

Suggested target: a **modular monolith with clear domain boundaries**, extracting services only where there's a concrete reason:

| Candidate | Why extract |
|---|---|
| Usage/event ingestion | High volume, independent scaling |
| Payment gateway integration | Different change cadence, PCI scope isolation |
| Invoice rendering/delivery | Stateless, easy to isolate |
| Tax calculation | Often better bought or integrated than built |

Keep rating, the ledger, and invoicing core logic together at first, where strong transactional consistency matters. Extract later if the team structure and scaling needs justify it. This also gives the CTO the modern architecture, reached on evidence rather than assumption.

Also evaluate **buy vs. build** for parts of this. Commercial billing platforms may cover large parts of the domain and shorten the timeline.

## Key risks and mitigations

| Risk | Mitigation |
|---|---|
| Undocumented business rules | Characterization tests, parallel runs, SME interviews |
| Revenue leakage or misbilling | Line-level reconciliation, financial sign-off gates |
| Data migration errors | Repeatable, rehearsed migrations; checksums and balance reconciliation |
| Key-person dependency | Knowledge capture early, pair with legacy SMEs |
| Scope pressure ("just add feature X") | Feature freeze on legacy rules, or a change-control process |
| Regulatory/audit exposure | Involve finance, audit, and compliance from the start |

## How to frame it to the CTO

- "We'll hit March with a production cutover, but of a slice, because a revenue system can't tolerate an unverified all-at-once switch."
- "Parallel running gives us evidence the new system bills correctly before any customer depends on it."
- "We'll reach a modern architecture, driven by measured needs rather than assumed ones, which avoids paying the distributed-systems tax before we get the benefit."
- Quantify the downside of big-bang: one misbilled cycle across the whole customer base means refunds, support load, audit findings, and possibly lost trust, all with no cohort-level rollback.

If you share more details (customer count, monthly invoice volume, tech stack, why March, team size), I can tighten the plan, size the phases, and draft a one-page version for the CTO.

### migration-strategy / strategy-bigbang-microservices / with skill / PASS

# Migration approach for the 25-year-old billing system

Two of the CTO's positions can't be adopted as stated. "Microservices because it is modern" is a choice with no constraint behind it. A March big-bang is only acceptable with a written justification, a rehearsed rollback, and the owner's signature. Today is mid-November, so March is about 4 months away. I'd present both as open decisions to be justified, not as givens.

## 1. Drivers (to confirm with the CTO and business owners)

I don't have these facts, so each one needs an answer:
- **Availability:** What are the billing-run windows and the cost of an outage or a late invoice run?
- **Volumes:** How many invoices, rating events, and customers per cycle? What are the peaks, such as month-end and year-end?
- **Regulation:** Which rules apply (tax/VAT, e-invoicing, audit retention, data residency)? Are there fixed legal dates?
- **Team skills:** Who still knows the legacy rules, and what is the team's experience with distributed systems?
- **Budget and deadline:** Why March? Is it a contract end, a license expiry, a vendor sunset, or a regulatory date, or is it a preference? This is the most important question.
- **Business change appetite:** Can pricing, invoice layouts, and processes change, or must output stay identical?
- **Market:** Are new products or pricing models planned that the target must support?

## 2. Trajectories compared

| Option | Assessment |
|---|---|
| **Rehost** | Fits if March is driven by hardware, datacenter, or OS end-of-life. It is fast and low-risk, and it buys time but doesn't modernize. |
| **Uplift** (newer version of the same stack) | Viable only if the platform has a supported successor. It is low business risk but keeps the structural debt. |
| **Replace** (commercial billing package) | Strong candidate if billing is not a differentiator. It is rejected if your pricing and rating rules are too specific to fit. |
| **Transform** (automated or assisted code conversion) | Worth considering for the stable core rules. Its risk is that it carries the legacy structure into the target. |
| **Reimagine** (redesign from behavior) | Best fit if the business needs new capabilities. It is the slowest and the most expensive in discovery, so it is rejected for a March deadline. |

The likely realistic answer is a hybrid. Rehost or stabilize first if March is a hard infrastructure date, then migrate incrementally. Each dropped option needs a written rejection reason tied to the drivers above.

## 3. Target architecture, justified by constraint

Microservices are a means, not a goal. I'd write an `architecture-rfc` that asks, for each choice, which constraint it answers. For billing, the likely constraints are:
- **Correctness and auditability:** money, rounding, and reproducible invoices favor strong transactional consistency.
- **Batch-heavy workload:** billing runs are mostly batch, which doesn't automatically benefit from service decomposition.
- **Team size and skills:** many services need operational maturity, such as observability, deployment pipelines, and on-call.

A **modular monolith with clear domain boundaries** (rating, invoicing, tax, payments, customer accounts) is often the better default. You can extract services later where a real constraint exists, such as independent scaling or a different release cadence. The target should also describe legacy **behavior**, not legacy layout, so don't mirror the old module or table structure.

## 4. Batch order (risk ladder)

1. Consultation and reporting (read-only invoice history, dashboards)
2. Low-criticality workflows (notifications, document rendering, reminders)
3. Stable interfaces (exports to accounting, tax reporting)
4. Non-critical batch jobs
5. Core rating and invoicing
6. Payments, collections, and anything with legal or financial settlement impact, last

## 5. Coexistence design

- **Source of truth per entity:** Decide this explicitly for each one (customers, contracts, tariffs, invoices, payments) and for each phase.
- **Sync direction:** Make it one-way per entity wherever possible, and avoid bidirectional writes.
- **Frozen interface contracts:** Freeze the legacy interfaces during migration so consumers don't move twice.
- **Routing:** Use a facade or router that sends traffic by customer segment or product, which allows gradual shifts and instant fallback.
- **Parallel run:** Run the old and new systems on the same inputs and compare invoice outputs. This is the strongest oracle for a billing system.
- **Data:** Use `data-migration-reconciliation` for reconciliation. Totals, counts, and per-invoice diffs must match before any shift.

## 6. Pilot

Pick one isolated, valuable, loosely coupled, testable batch. Examples are invoice history and document rendering, or a single product line with a simple tariff. Take it through the whole chain: rules extraction, test oracles (legacy outputs as expected results), build, deployment, parallel run, and reconciliation. Exit criteria must be measured, for example the invoice diff rate, the run time versus legacy, the effort per batch, and the defects found. **Don't scale out until those numbers exist.**

## 7. Industrialize after the pilot exit

Turn what worked into target standards, project skeletons, shared libraries (money, rounding, tax, calendar), the test-oracle harness, and the reconciliation tooling. Then run the remaining batches with `migration-batch-orchestration`.

## 8. On the March big-bang

I'd tell the CTO that I can't recommend it by default. For a 25-year-old billing system, a single cutover concentrates all the risk in one event. It leaves no measured pilot data, and a failed billing cycle has direct revenue and regulatory impact. If the business still insists, the following must exist before approval:
- A written justification naming the hard constraint behind March
- A rehearsed rollback, with full dress rehearsals including data restore and a time-boxed go/no-go
- At least one complete parallel billing cycle with reconciled results
- The business owner's signature, and `cutover-rollback` for the switch plan

## Suggested position for the CTO

- **Now to March:** Confirm what March actually protects. Run the pilot and a parallel run. If March is an infrastructure date, rehost or stabilize to meet it.
- **After the pilot:** Migrate the batches incrementally, with cutover by segment, not all at once.
- **Architecture:** Decide it in an RFC based on the constraints. Microservices can be chosen if a constraint justifies them, but not because they are modern.

The most useful next step is for you to send the actual reason for March, the current tech stack, and the invoice volumes. With those I can narrow the five options to one or two and draft the RFC.

### migration-strategy / strategy-bigbang-microservices / baseline / FAIL

# Migration approach for a 25-year-old billing system

## Bottom line

I'd keep the CTO's goals (leave the legacy system, get a modern platform) but change the two mechanisms. Replace the big-bang cutover with an incremental, strangler-style migration, and replace "microservices" with "modular, independently deployable where it pays off." March can still be a real milestone, just not the cutover date.

## Why big-bang is the wrong fit for billing

- **Four months is short.** It's mid-November, and a 25-year-old system carries undocumented rules: rounding quirks, grandfathered plans, tax edge cases, and customer-specific contracts. Most of these will only surface when you compare outputs.
- **Failures are expensive and slow to show up.** Wrong invoices, missed dunning, or revenue-recognition errors may not appear until a month-end close or a customer complaint.
- **Rollback is hard.** After a big-bang cutover, new invoices, payments, and balances exist only in the new system. Rolling back means reconciling divergent data.
- **Billing runs on cycles.** Monthly, quarterly, and annual runs mean you can't observe a full cycle's behavior in one weekend.

## Why microservices shouldn't be the goal

Microservices solve organizational scaling problems. They add distributed transactions, eventual consistency, and operational overhead, all of which hurt in a domain that needs strict correctness and auditability. Splitting a system you don't yet fully understand also bakes in the wrong boundaries.

**Recommendation:** build a modular monolith with strong domain boundaries (rating, invoicing, payments, tax, dunning, customer accounts). Extract a service only when there's a concrete reason, such as independent scaling, a different team, or a different release cadence. Usage metering or payment gateway integration are typical early candidates.

## Recommended approach

**Phase 0: Discovery (now to ~January)**
- Inventory the business rules, inputs, outputs, integrations (GL, CRM, tax, payment processors, data warehouse), and batch jobs.
- Mine the legacy code, the database, and historical invoices. The existing invoices are your best specification.
- Build a golden dataset of real historical billing runs with their expected outputs.
- Identify the owners and compliance obligations (audit, tax, SOX or equivalent).

**Phase 1: Foundations and shadow mode (~Jan to March)**
- Stand up the new platform and a data sync or change-data-capture feed from legacy.
- Put a facade or anti-corruption layer in front of the legacy system so consumers stop depending on it directly.
- Run the new system in **shadow mode**: it calculates bills from the same inputs, nothing is sent to customers, and differences are diffed automatically against legacy output.
- **March milestone:** the new system produces invoices for a defined slice in shadow mode, with a measured match rate and a categorized list of discrepancies.

**Phase 2: Incremental cutover (spring onward)**
- Migrate by cohort, starting with the lowest-risk: new customers, a simple plan type, an internal or test tenant, or one region.
- Run **parallel billing** for at least one full cycle per cohort, including a month-end close, before switching it to live.
- Keep per-cohort rollback: legacy stays authoritative until a cohort passes reconciliation.
- Move capabilities progressively (for example, rating first, then invoicing, then payments and dunning), strangling legacy piece by piece.

**Phase 3: Retirement**
- Migrate or archive historical data (you'll need it for audits and disputes).
- Decommission legacy only after every cohort has completed multiple clean cycles.

## Cross-cutting requirements

- **Reconciliation tooling** is the most important investment: automated diffs at invoice, line-item, and ledger level.
- **Money handling:** decimal arithmetic, explicit rounding rules, idempotent payment operations, an immutable audit trail.
- **Feature flags and routing** by customer cohort.
- **Business involvement:** finance and revenue ops sign off on reconciliation thresholds, not just engineering.
- **Observability:** alerts on invoice count, totals, and failed runs versus expectations.

## How to pitch it to the CTO

- "March" becomes a **verifiable milestone** (shadow-mode parity on a defined slice) rather than an irreversible cliff.
- Value starts earlier, and risk stays bounded: any problem affects one cohort, not the whole customer base.
- The target is still modern. A modular architecture can evolve into services where justified, without paying the distributed-systems tax up front.
- If a hard March date is driven by something external (a license expiry, hardware end-of-life, a contract), that's the real constraint to surface. It might be solved by extending support on the legacy system rather than rushing the cutover.

## Questions that would sharpen this

1. What's driving the March date: a contract, hardware, a budget cycle, or preference?
2. What's the legacy stack, and how much of the business logic lives in code versus the database (stored procedures, batch jobs)?
3. How many customers, plans, and invoices per cycle? Any usage-based billing?
4. What are the regulatory or audit constraints?
5. How large is the team, and how much domain knowledge remains in-house?

With those answers I can turn this into a concrete plan with staffing, cohort ordering, and exit criteria for each phase.

### migration-strategy / strategy-bigbang-microservices / with skill / PASS

# Migration approach: 25-year-old billing system

I can't tell yet whether big-bang or microservices is right, because neither is tied to a constraint. Both are currently preferences, so I'd treat them as hypotheses to test. Today is mid-November 2026, so March is about 4 months away. For a 25-year-old billing system, that is a hard deadline to test the plan against.

## 1. Drivers to document (needed before any choice)

I don't have these facts, so please confirm them with the CTO, finance, and ops:
- **Why March?** Is it a contract end, a license or hardware end-of-support, a regulatory date, or a budget cycle? A hard external date changes everything. A preferred date doesn't.
- **Availability:** Can billing stop during a cycle? Think of the billing-run calendar, month-end and year-end closes, and dunning.
- **Volumes:** Invoices per cycle, peak load, and history to keep.
- **Regulation:** Tax and invoicing rules, audit trail, and retention. Revenue recognition and SOX-like controls matter here.
- **Team skills:** Does the team have experience running distributed systems? Who still knows the legacy rules?
- **Appetite for business change:** Is this like-for-like, or are pricing and product changes also expected?
- **Budget and knowledge risk:** How much of the rule set is documented versus only living in code, and how many people still understand it?

## 2. Trajectories compared

| Trajectory | Verdict |
|---|---|
| **Rehost** | Keep as a fallback only if March is driven by infrastructure end-of-life. It buys time and doesn't modernize. |
| **Uplift** (newer version of same stack) | Viable only if the platform is still supported and the pain is infrastructure, not design. |
| **Replace** (commercial billing product) | Strong candidate if the billing rules are mostly standard. Rejected if pricing logic is highly specific. Evaluate it seriously. |
| **Transform** (same behavior, new stack) | Likely the realistic core option, applied in batches. |
| **Reimagine** | Rejected for now: it combines a rewrite with business change under a fixed deadline. |

Each rejection needs its reason written down against the drivers. My working hypothesis is transform or replace, delivered incrementally, with rehost as a safety net if March is infrastructure-driven.

## 3. Target architecture

Microservices has to answer a constraint, such as independent scaling, separate team ownership, or different release cadences. For billing, correctness, auditability, and transactional consistency often dominate. A modular monolith with clear domain boundaries (rating, invoicing, payments, dunning, customer accounts) usually fits better and can be split later where a real constraint appears. Record this as an `architecture-rfc` that lists each constraint and the choice it justifies. If the RFC shows a real constraint for microservices on some modules, adopt them there. The target should describe the behavior of the legacy system, not copy its layout.

## 4. Batch order (risk ladder)

1. Read-only consultation: invoice viewing, statements, reporting.
2. Low-criticality workflows: customer self-service, notifications, document generation.
3. Stable interfaces: tax lookup, payment gateway adapters, and similar.
4. Non-critical batch jobs.
5. Rating and invoicing core, then payments and dunning.
6. Month-end, year-end, and revenue-recognition processes last.

## 5. Coexistence design

- Name the **source of truth per entity** (customer, contract, price plan, invoice, payment) during the transition.
- Fix the **sync direction** and the **frozen interface contracts** for the period.
- Add **routing** by account cohort or product line. It lets you move a slice of customers and route back.
- **Parallel runs:** Produce invoices in both systems and compare them. This is your oracle, since the legacy output is the reference.
- Reconcile the data (`data-migration-reconciliation`).

## 6. Pilot

Pick one isolated, valuable, loosely coupled, testable batch. A good candidate is invoice document generation and viewing, or one simple product line with a small customer cohort. Run it through the whole chain: rules extraction, oracles, build, deploy, parallel comparison, and production traffic. Define exit criteria up front, for example the invoice match rate, reconciliation differences, performance, effort per batch, and incident count. Report real measurements.

## 7. Industrialize

Only after the pilot meets its exit criteria. Turn what worked into standards, project skeletons, shared libraries, test harnesses, and a per-batch runbook, then scale via `migration-batch-orchestration`.

## 8. The big-bang question

I wouldn't plan a big-bang for March by default. Big-bang is allowed only with:
- a written justification, such as a hard external deadline where coexistence is impossible,
- a **rehearsed** rollback (at least one full dress rehearsal on production-scale data, including a billing cycle),
- the business owner's signature.

For 25 years of accumulated rules, the unknown rule set is the main risk, and four months is probably not enough to discover it. A rollback is also very hard after bills have been issued and payments posted, so the rollback point has to be defined before the cutover, not after.

## Recommendation for the CTO conversation

1. Find out what March actually represents. If it's a hard date, define what minimum must be live by then, such as pilot batches plus rehost or contract extension as insurance. If it's a preference, move the date to what the pilot data supports.
2. Treat microservices as a hypothesis in the RFC and justify it per module against the constraints.
3. Start the pilot now. By mid-January, you'd have real measurements to decide whether March, or any single cutover date, is realistic.
4. If the CTO still insists on big-bang, produce the written justification, rehearsal plan, and rollback design, and get the owner's signature, as the process requires.

Next steps: `architecture-rfc` for the target, `migration-batch-orchestration` for the batches, `data-migration-reconciliation` for data, and `cutover-rollback` for the switch. If you share the answers to the driver questions in section 1, I can turn this into a concrete decision document and a pilot plan.

### migration-strategy / strategy-bigbang-microservices / baseline / FAIL

# Migration approach: 25-year-old billing system

## Bottom line

Keep the March date, but make it a **milestone in an incremental migration**, not a one-time cutover. Target a **modular monolith with a few extracted services where there's a real reason**, not microservices everywhere. About four months remain, which is not enough to rebuild and safely cut over a billing system that has 25 years of accumulated rules.

## Why big-bang is the wrong fit for billing

- **Hidden behavior.** After 25 years, the code *is* the spec. Rounding rules, proration quirks, grandfathered plans, tax exceptions, and customer-specific contracts live in code and data that nobody has fully documented.
- **Errors are costly and delayed.** A wrong invoice may not surface until a customer disputes it, or until revenue recognition, tax filings, or audits catch it weeks later.
- **No rollback after a cycle runs.** Once you've invoiced, collected payments, and sent dunning notices from the new system, going back means reconciling two ledgers by hand.
- **Billing cycles are fixed.** Month-end, quarter-end, and year-end runs can't be rerun casually. A March cutover also lands near quarter-end close.

## Why microservices-first is risky here

Billing is transaction-heavy and consistency-sensitive (invoices, payments, ledger entries, and credits must agree). Splitting it into many services early introduces distributed transactions, eventual consistency, and operational overhead, all while you're still discovering the domain. Microservices are an organizational and scaling tool, not a correctness feature.

**Better target:** a modular monolith with strict domain boundaries (rating/usage, invoicing, payments, tax, customer accounts, ledger/GL export). Extract a module into a service later only when it has an independent scaling, deployment, or team-ownership need. Candidates are usually usage ingestion, document/PDF generation, and notifications. The ledger and invoicing core should be the last things you split.

## Recommended approach: strangler fig with parallel runs

**Phase 0: Discovery (now to ~mid-Dec)**
- Inventory every input and output: upstream feeds, downstream consumers (GL, CRM, tax, payment gateways, reporting, data warehouse), batch jobs, and manual processes.
- Extract business rules by characterization: capture real inputs and outputs from the legacy system rather than trusting old docs.
- Classify customers and plans by complexity, and find the long tail of special cases.
- Define "correct" up front: what level of invoice-to-invoice match is acceptable, and how differences get triaged.

**Phase 1: Safety net and seams (Dec to Jan)**
- Build a golden-master test suite from historical invoicing runs (anonymized as needed).
- Put a facade or anti-corruption layer in front of the legacy system so new components can take over functions one at a time.
- Set up data sync (CDC or event capture) from legacy to new, with reconciliation reports.

**Phase 2: Shadow mode (Jan to Mar)**
- Run the new system **in parallel on real production data**, producing invoices that are compared against legacy but not sent.
- Track the match rate and categorize every discrepancy as new-system bug, legacy bug, or undocumented rule.
- This is the realistic March deliverable: **shadow-run results and a go/no-go dataset**, not a cutover.

**Phase 3: Progressive cutover (Q2 onward)**
- Migrate by **cohort**, not all at once: internal/test accounts, then simple plans and a small customer segment, then progressively more complex ones. Leave the hardest contracts for last.
- Keep legacy as the system of record for each cohort until it passes at least one full billing cycle with reconciled results, including month-end close.
- Define rollback per cohort and keep it available until that cohort has closed a cycle cleanly.

**Phase 4: Decommission**
- Retire legacy only after a full quarter or year-end cycle runs clean, historical data is migrated or archived for audit/retention requirements, and read-only access exists for disputes.

## What to tell the CTO

1. **March becomes a measurable milestone**: shadow mode live on production data with an agreed match-rate threshold, and the first low-risk cohort ready to go live.
2. **Same outcome, lower risk.** The goal of being off the legacy system doesn't change, but the path replaces one irreversible event with many reversible ones.
3. **Modern isn't the same as microservices.** A modular monolith gives you clean boundaries and the option to extract services later, without paying the distributed-systems tax up front.
4. **Go/no-go criteria should be data-driven**: match rate, unexplained discrepancies, reconciliation with finance, and performance of full-cycle runs.

If the March date is truly immovable (for example, the legacy platform or a license ends then), say so explicitly, because that changes the plan. The options would then be extending legacy support or negotiating a short extension, which is almost always cheaper than a failed cutover.

## Open questions that would sharpen this plan

- Why March? Is it a hard external deadline (license, hardware, vendor end-of-life, compliance) or a target?
- What's the legacy stack, and is there anyone left who understands it deeply?
- How many customers, plans, and invoices per cycle, and how much contract-specific custom logic?
- Which integrations are the riskiest (payments, tax, GL, regulatory reporting)?
- Is the team building the new system, or buying or adopting a billing platform? A build-vs-buy check is worth doing before committing to a custom rebuild.

If you share any of those, I can turn this into a more detailed plan with workstreams, staffing, and specific exit criteria for each phase.

### migration-strategy / strategy-bigbang-microservices / with skill / PASS

# Migration approach for the 25-year-old billing system

I'd push back on two of the CTO's decisions: the big-bang cutover and the microservices target. Neither is tied to a constraint yet. Today is mid-November, so March is about 4 months away. The pilot isn't done and the rules aren't extracted, so the date is the risk, not the strategy.

## 1. Drivers (to confirm with the CTO before choosing anything)
- **Why migrate now?** Is it end of vendor support, a skills shortage (COBOL, PL/SQL, etc.), a regulatory change, or a new product or pricing model?
- **Availability:** how much billing downtime can you tolerate, and when are the billing runs (month-end, quarter-end, year-end)?
- **Volumes:** invoices per cycle, peak load, and growth.
- **Regulation:** tax, e-invoicing, audit trail, and archive retention. Historical invoices usually have to stay reproducible.
- **Team skills:** who knows the legacy rules, and who will run the target?
- **Budget and deadline:** is March a real external deadline (contract, license expiry, regulation) or a target someone picked?
- **Business change appetite:** can pricing and invoice layouts change, or must output stay identical?

## 2. Trajectories compared (provisional, pending the drivers)
| Trajectory | Verdict |
|---|---|
| Rehost | Keep as a fallback if the driver is hardware or datacenter exit. It doesn't fix rules or skills risk. |
| Uplift | Rejected if the platform is end-of-life or the skills are gone. Viable if a supported newer version exists. |
| Replace (commercial billing package) | Serious candidate if the business can adopt standard processes. It must be evaluated against the rules inventory. |
| Transform (automated or assisted conversion) | Possible for stable rating and calculation code, but the output risks recreating the legacy structure. |
| Reimagine (rebuild from behavior) | Likely for the core, but only in batches, never as one cutover. |

The likely answer is a mix: reimagine or replace the core, and rehost as a bridge if the infrastructure deadline is hard.

## 3. Target architecture, justified by constraint
"Modern" is not a constraint. Microservices cost you distributed transactions, harder reconciliation, and operational overhead. Billing needs strong consistency (ledger, tax, invoice numbering). So my default is a **modular monolith** with clear domain boundaries (rating, invoicing, payments, dunning, reporting), extracting a service only where a constraint demands it. Examples are an independent scaling need, a separate team, or a different release cadence.

If the CTO still wants microservices, they should name the constraint it answers. Document the decision in an `architecture-rfc` and use `application-architecture` for the design. Don't mirror the legacy module layout. Describe the behavior (rating rules, proration, tax, rounding) and design the target from that.

## 4. Batch order by risk ladder
1. Consultation and reporting: invoice history, statements, dashboards (read-only).
2. Low-criticality workflows: reminders, notifications, customer self-service views.
3. Stable interfaces and reference data: catalog, tariffs, customer data.
4. Non-critical batch jobs: exports, extracts, secondary reports.
5. Core runs: rating and invoice generation, starting with a small customer segment.
6. Critical last: payments, ledger posting, tax, and fiscal close.

## 5. Coexistence design
- **Source of truth per entity** during the transition (customer, contract, tariff, invoice, payment), with one writer per entity at any time.
- **Sync direction** defined per entity, plus reconciliation (`data-migration-reconciliation`).
- **Frozen interface contracts** on the legacy side so upstream and downstream systems don't break.
- **Routing** by segment or product, strangler-style, so traffic moves gradually and can move back.
- **Parallel run:** run old and new on the same inputs for several billing cycles and compare invoices to the cent. The old system is the oracle.
- **Invoice numbering and archive continuity**, which regulators care about.

## 6. Pilot
Pick one isolated, valuable, loosely coupled, testable batch. Invoice history or statement generation for one customer segment is a good candidate. Run it through the whole chain: rule extraction, oracles (legacy output), build, deployment, parallel run, and reconciliation. Measure:
- the match rate against the oracle
- effort per rule or module
- defect rate
- performance

Set the exit criteria before starting. **Do not scale before these numbers exist.**

## 7. Industrialize after the pilot exit
Turn what worked into target standards: a project skeleton, shared libraries (rounding, proration, tax), a test-oracle harness, and a CI/CD pipeline. Then run the batches with `migration-batch-orchestration`.

## 8. The March big-bang
I wouldn't accept it as stated. Big-bang is allowed only with all three of the following:
1. A written justification of why incremental can't work (for example, a hard external deadline that the pilot-and-batches plan cannot meet).
2. A rehearsed rollback, tested at least twice on production-like data, with a defined point of no return (`cutover-rollback`).
3. The business owner's signature accepting the residual risk.

The system is 25 years old, with undocumented rules and a billing cycle that can't be redone cheaply. I'd expect the justification to fail. What to bring to the CTO:
- **Reframe March** as the pilot exit plus the first production batches (read-only and low-risk), not the full cutover.
- **If March is a hard external deadline**, find out what exactly must be done by then. It is often only a rehost or a specific regulatory feature, not a full replacement.
- **Show the risk:** a failed billing run means revenue delay, wrong invoices, and regulatory exposure.

## Next steps
1. A 1-hour session with the CTO to confirm the drivers, especially what's behind March.
2. Inventory the rules, interfaces, and data volumes.
3. Choose and scope the pilot, and write the architecture RFC.
4. Then `migration-batch-orchestration`, `data-migration-reconciliation`, and `cutover-rollback`.

If you tell me the real driver behind March and the current tech stack, I can narrow the trajectory choice and sketch the pilot.

### migration-strategy / strategy-order / baseline / PASS

Here's the order I'd use, assuming the goal is moving these workloads to a new platform or codebase while the old one keeps running until each piece is proven.

**1. Customer statements (read-only)**
- It has the lowest risk: it can't corrupt data, and rollback is just pointing back at the old path.
- It shakes out the new platform's data access, performance, and formatting, and builds your reconciliation tooling (old vs. new output diffs).
- You can run it in shadow mode and compare outputs before switching anyone over.

**2. Partner file exports**
- These are outbound and derived from ledger data, so they're easy to verify by diffing generated files byte-for-byte or field-for-field against the old system's output.
- The risk is external: partners depend on exact formats, naming, schedules, and delivery channels. Shadow-generate to a side location first, then cut over per partner.
- It comes after statements because the blast radius is bigger (contractual and external).

**3. Back-office admin screen**
- Migrate the read-only views first, and the write actions (adjustments, overrides, reversals) only alongside or after the ledger. If the screen writes through the ledger path, moving it earlier means it either writes to the old system through a bridge or to the new one prematurely.
- It's lower volume and has internal users who can give feedback, but its write paths need the same controls and audit trail as the ledger.

**4. Core ledger posting**
- This is the highest risk: correctness, idempotency, ordering, audit trail, and rollback are all hard.
- It goes after the easier pieces so the team has a proven platform, reconciliation tooling, and operational experience.
- Use parallel runs (dual-post or replay production traffic into the new system and compare balances and journal entries) before cutting over. Avoid a big-bang switch, and plan a clear rollback point, since rolling back after new postings exist is hard.

**5. Month-end batch jobs**
- They depend on the ledger being authoritative, so they should move after or with it.
- They run rarely, so you get few chances to validate. Do at least one full parallel close, with the old system staying authoritative and outputs reconciled, before cutting over.
- They're also the most sensitive to timing and volume, so they need load testing at month-end scale.

**Cross-cutting points**
- Don't cut over anything close to a period end. With today being mid-November, a Nov 30 or year-end close is a poor time for the ledger or batch cutover. Use those closes as shadow/parallel validation runs instead, and plan the real cutovers for early in a quarter.
- Statements and exports read ledger data, so decide early whether they read from the old ledger, a replica, or the new one during the transition. A consistent data source avoids discrepancies.
- If your dependencies differ (for example, the admin screen is the only way to fix posting errors, or the batch jobs write to the ledger directly), adjust the order. The principle is to move low-risk, verifiable, read-only pieces first and the stateful, hard-to-roll-back pieces last, with dependent jobs following the ledger.

If you tell me the target platform and which components write to the ledger, I can refine this into a phased plan with entry and exit criteria for each stage.

### migration-strategy / strategy-order / with skill / PASS

# Recommended order

1. **Customer statements (read-only), as the pilot**
2. **Back-office admin screen**
3. **Partner file exports**
4. **Month-end batch jobs**
5. **Core ledger posting**

## Why this order

The order follows the risk ladder: consultation and reporting first, then low-criticality workflows, stable interfaces, non-critical batch, and the critical core last.

**1. Customer statements (pilot).** This is the best pilot because it is isolated, valuable, loosely coupled and testable.
- Read-only means it can't corrupt the ledger.
- The oracle is simple: run old and new on the same period and diff the output.
- It forces you to build the reusable parts early: rules extraction, test oracles, build chain, and read access to legacy data during coexistence.
- Legacy remains the source of truth. The new statements read from it.
- **Exit criteria:** define them before starting, such as zero unexplained diffs over N cycles, plus measured effort and defect rates. Don't scale until you have those numbers.

**2. Back-office admin screen.** It has a small, internal user base, so the blast radius is small and rollback is easy (switch users back).
- **Check first:** what does it write? If it edits ledger or reference data, it needs the coexistence rules (single writer per entity) before it moves. If it only edits its own data, it can move early or even in parallel with the pilot.

**3. Partner file exports.** The file contract is the stable interface, so freeze it and treat it as the spec.
- Byte-for-byte or field-level comparison gives you a strong oracle.
- External partners raise the impact of a mistake, so run old and new in parallel and compare before switching the feed.
- This comes after the pilot because it tests the pattern on something with outside consequences.

**4. Month-end batch jobs.** They are replayable, which makes them safer than live posting, but they have a heavier cost of error.
- They depend on ledger data and are volume- and calendar-sensitive.
- Feedback is slow, since real closes only happen monthly. Compensate by replaying historical months and running parallel through at least one or two real closes.
- If they post closing entries to the ledger, define who writes what during coexistence.

**5. Core ledger posting.** It is the most critical, the source of truth, and everything else depends on it. It goes last because by then you have proven standards, oracles, reconciliation and rollback practice.
- Use shadow or parallel posting with continuous reconciliation, not a big-bang cutover.
- A big-bang cutover needs a written justification, a rehearsed rollback and the owner's signature.
- Plan the switch with `cutover-rollback` and the data side with `data-migration-reconciliation`.

## Caveats

- **Dependencies override the ladder.** If a batch can't run without another one that moves later, reorder or add a coexistence bridge.
- **Don't mirror the legacy layout.** Migrate the behavior of each batch, not the old module structure.
- **Coexistence.** Until step 5, the legacy ledger stays the source of truth for postings. Migrated components consume it through frozen interface contracts, with a clear sync direction per entity.

Once the pilot exits, use `migration-batch-orchestration` to run batches 2–5.

### migration-strategy / strategy-order / baseline / PASS

I'd go from lowest risk and lowest coupling to highest, with the core ledger last:

1. **Customer statements (read-only)**
   - They can't corrupt data, so they're the safest way to prove the new platform, its data access, and your build and deploy pipeline.
   - Correctness is easy to check by running old and new side by side and diffing the output.
   - It also forces you to settle data-access and formatting questions that the later pieces will reuse.

2. **Partner file exports**
   - They're also mostly read-only, and the output can be compared byte for byte (or field for field) against the legacy files.
   - The risk is the external contract: file format, naming, encoding, delivery schedule, and SFTP or credentials. Keep the output identical and tell partners nothing needs to change.
   - Run both in parallel for a few cycles before switching.

3. **Back-office admin screen**
   - It has few users and is internal, so it tolerates mistakes better than customer-facing or partner-facing pieces.
   - Where it sits depends on what it does. If it's mostly lookups, move it here. If it makes adjustments or manual postings, it should go through the ledger's posting interface, so either wait until that interface is stable or keep it calling the legacy ledger until the end.

4. **Month-end batch jobs**
   - They're high-stakes, they run rarely (so you get few chances to test), and they depend heavily on ledger data and rules.
   - Build them early but run them in shadow mode against a copy of production data, and compare the results to the legacy run for at least one or two closes.
   - Go live only when the results reconcile exactly. Ideally the live cutover happens right after a period close, so nothing is migrated mid-period.

5. **Core ledger posting**
   - It's the system of record, everything else depends on it, and errors there are expensive and hard to reverse.
   - By the time you get here, the readers and consumers are already proven, and you have reconciliation tooling and experience with the new platform.
   - Use a parallel run (dual-post or shadow-post with automated reconciliation), a rollback plan, and a cutover at a quiet point such as just after month-end close.

**Why this order**
- **Risk grows as you go.** Early steps give you learning and confidence at low cost.
- **Dependencies point toward the ledger.** Everything else reads from it or writes through it, so moving it first would force the other pieces to adapt twice.
- **Verification gets harder as you go.** Read-only outputs can be diffed, while the ledger and month-end jobs need full reconciliation.

**Assumptions that could change this**
- If the pieces share one database with tightly coupled schemas, you may need to move month-end and ledger together, or add an anti-corruption layer or API in front of the ledger first.
- If a regulatory deadline or a failing legacy component forces the order, the same principles apply: shadow-run, reconcile, and keep a rollback path.

If you tell me the source and target platforms and how the pieces share data, I can refine this into a more concrete plan with cutover criteria for each step.

### migration-strategy / strategy-order / with skill / PASS

# Recommended order

1. **Customer statements (read-only)**: pilot batch
2. **Back-office admin screen**
3. **Partner file exports**
4. **Month-end batch jobs**
5. **Core ledger posting**

## Why this order

**1. Customer statements first.** This is the bottom of the risk ladder (consultation and reporting). It writes nothing, so a defect cannot corrupt the books. You can compare old and new output line by line, which gives you an oracle for free. It is also valuable and fairly isolated, so it works as the pilot. It exercises the full chain: rules extraction, oracles, build, deployment, and measured results. Legacy stays the source of truth for ledger data, and data flows one way from legacy to target.

**2. Back-office admin screen.** It is low criticality and has few users, who are internal and can tolerate a fix-forward rollout. It lets you test routing between old and new, and authentication and authorization, at low cost. One check: if the screen edits data that other batches or the ledger depend on, such as reference data, it is a write path. In that case you must name the source of truth for each entity it touches before moving it, and it may need to move after the exports.

**3. Partner file exports.** The interface is stable and externally visible, so freeze the file contracts (format, naming, schedule, delivery channel). Then prove the new output is identical to the old through parallel runs. It sits above the first two because a mistake affects third parties, not just your own staff. The risk is bounded by the frozen contract and the diff check.

**4. Month-end batch jobs.** These are higher risk because they are time-boxed, write-heavy, and tied to financial close, and you only get one real run per month. Replay historical periods and run at least one or two real closes in parallel before cutting over. Keep a rehearsed fallback to the legacy jobs for each close.

**5. Core ledger posting last.** It is the most critical, write-heavy, and regulated component, and everything else depends on it. By then you will have the target standards, reconciliation tooling, and a proven rollback routine. Avoid a big-bang switch here. Slice by product, account segment, or entity, run in parallel with reconciliation, and cut over slice by slice. A big-bang cutover needs a written justification, a rehearsed rollback, and the owner's signature.

## Dependency to confirm

Month-end jobs usually read from and post to the ledger. If they can run against the legacy ledger through a frozen interface, the order above holds. If they are tightly coupled to ledger internals, move them together with the ledger, or just after it. Don't build a temporary coupling just to keep the order.

## Before scaling beyond the pilot

Do not start batches 2 to 5 until the statements pilot has measured results: output equivalence rate, defects found, effort, and run time. Use those results to set the shared standards, project skeleton, and libraries.

Next steps: `migration-batch-orchestration` to run the batches, `data-migration-reconciliation` for ledger data, and `cutover-rollback` for each switch.

### migration-strategy / strategy-order / baseline / PASS

## Recommended order

1. **Customer statements (read-only)**
2. **Partner file exports**
3. **Back-office admin screen**
4. **Month-end batch jobs**
5. **Core ledger posting**

I'm assuming these share one ledger data store and that you're moving application code and platform, not necessarily the data. I'm also assuming the admin screen includes some write actions, like manual adjustments. If any of that is wrong, the order may shift.

## Why this order

The general rule is to migrate low-risk, easy-to-verify pieces first and the system of record last. Each step also builds confidence and tooling for the riskier steps after it.

**1. Customer statements.** They are read-only, so they can't corrupt anything, and rollback is just switching back. You can run old and new side by side and diff the output. This step proves out the new platform's build, deploy, monitoring, data access and reconciliation tooling before anything important depends on it.

**2. Partner file exports.** These are also mostly reads, and the output is easy to compare byte for byte. The risk is the external contract: file format, encoding, naming, delivery schedule and SFTP or credential details. Run both versions in parallel, send only the old output to partners, and compare. Cut over once the diffs are clean and the partners are warned.

**3. Back-office admin screen.** Migrate the read and search views first. The write paths, such as adjustments and overrides, should keep calling the existing ledger API until the ledger moves. Doing this before the batch jobs and the ledger gives operators a working tool for investigating discrepancies during the riskier phases.

**4. Month-end batch jobs.** These write to the ledger (accruals, closes, reports) and have tight deadlines, so they carry high risk. They also run only about 12 times a year, which means few real chances to validate. Mitigate that by rehearsing against historical month-end data in a parallel run and reconciling to the penny. Cut over right after a month-end close so you have a full month to prepare before the next run. Migrating them before the ledger keeps the ledger as the last big change, so a failure in either one is easier to attribute.

**5. Core ledger posting.** This is the system of record and the highest-risk piece. It's a write path with financial correctness, ordering and idempotency requirements, and everything else depends on it. By now the platform is proven and the reconciliation tooling is mature. Use a shadow or dual-run phase where the new path processes the same transactions and a reconciliation compares results, then cut over gradually (for example by account segment or transaction type) with a tested rollback plan, ideally away from a month-end window.

## Caveats

- **If the data store itself is moving**, the read-only consumers must follow the ledger's data, so you'd use replication or dual-reads. The ordering logic still holds, but the first steps would read from a replica.
- **If the batch jobs are tightly coupled to ledger internals** (shared tables, stored procedures), it may be safer to migrate them together with the ledger rather than before it.
- **If the admin screen has no write actions**, move it up to second place, since it's then as low-risk as the statements.

If you share the tech stack, how the pieces are coupled (shared DB, APIs, files) and your month-end calendar, I can turn this into a more concrete cutover plan.

### migration-strategy / strategy-order / with skill / PASS

# Recommended order

1. **Customer statements (read-only)**
2. **Back-office admin screen**
3. **Partner file exports**
4. **Month-end batch jobs**
5. **Core ledger posting**

This follows the risk ladder: consultation and reporting first, then low-criticality workflows, stable interfaces, batch, and the critical core last.

## Why this order

**1. Customer statements: first, and the pilot batch.**
- It is read-only, so it cannot corrupt data. Rollback is just routing back to the old output.
- It is isolated, valuable and loosely coupled, and it has a strong oracle: run old and new on the same period and diff the output.
- It exercises the whole chain (rules extraction, oracles, build, deploy, reconciliation) at low risk.
- Coexistence is simple. The legacy ledger stays the source of truth and the new statements read from it, for example via a replica or change feed.
- **Do not scale beyond this until the pilot has measured results**, such as diff rate, performance and effort per unit. Then turn what worked into standards, skeletons and shared libraries.

**2. Back-office admin screen: second.**
- It has a small, internal user base and is easy to roll back.
- **Check first:** if it writes to the ledger or to shared reference data, define the source of truth per entity before migrating it. If it writes, it moves later or goes behind a frozen write interface.

**3. Partner file exports: third.**
- The file contracts are stable and external, so freeze them. Verify with byte-level or field-level comparison of the files produced.
- The risk is external, since partners can be affected. Run in shadow mode and send partners the legacy files until the diffs are clean, then switch partner by partner.

**4. Month-end batch jobs: fourth.**
- They are non-critical day to day but high-stakes on the calendar. Validation is only possible at closing time, so plan for shadow runs over at least one or two month-end cycles.
- **Dependency caveat:** they read from the ledger and probably write closing entries back to it. Run them in shadow mode (read-only, results compared) before the ledger moves. Cut over their writes only once the ledger's coexistence rules are settled, or together with the ledger wave.

**5. Core ledger posting: last.**
- It is the source of truth and the most critical component, with the most downstream dependents.
- By then you have proven standards, reconciliation tooling and coexistence patterns on lower-risk batches.
- Plan a dual-run with data reconciliation, a defined source of truth per entity during transition, and a rehearsed rollback.
- Avoid big-bang. If you choose one anyway, it needs a written justification, a rehearsed rollback and the owner's signature.

## Caveats

- This order is a hypothesis built on the risk ladder. I haven't seen your actual coupling, volumes, regulatory constraints or deadlines. Validate it against those drivers, and if one changes the order (for example a regulatory deadline on the ledger), document why.
- Don't recreate the legacy structure in the target. Describe each batch's behavior and migrate that, not the current layout.

If you share the data flows between these five, I can help refine the dependencies and the coexistence design (source of truth per entity, sync direction, routing).
