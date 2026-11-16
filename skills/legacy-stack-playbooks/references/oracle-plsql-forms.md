# Oracle PL/SQL, Forms, and Reports

Status: unvalidated against a live system. Dictionary view names vary by edition and privileges.

## Inventory sources

- Code: packages, procedures, functions, triggers, and views from the data dictionary source and dependency views; Forms and Reports modules exported to their XML form.
- Entry points: scheduler jobs, database links, external tables, message queues, application server endpoints, Forms menus.
- Usage: workload repository statistics, SQL statistics history, table modification tracking, audit trails.

## Hidden behavior to turn into rules or questions

- Business rules living in triggers and Forms triggers; trigger firing order and cascades.
- The empty string equals null; three-valued logic; implicit type conversions and NLS settings (dates, decimal separators, sort order).
- DATE carries a time part; sequences may have gaps; row ordering without an explicit order clause.
- Commits inside procedures, autonomous transactions, and locking behavior with the default isolation level.
- Dynamic SQL and names resolved at run time, which the dependency views cannot see.
- Numeric precision of NUMBER without scale; rounding functions and their tie behavior.

## Oracle design

Run the same inputs through the packages in a test schema and through the target; compare table states, output files, and messages. Cover null, empty string, time-part, and locale cases explicitly.
