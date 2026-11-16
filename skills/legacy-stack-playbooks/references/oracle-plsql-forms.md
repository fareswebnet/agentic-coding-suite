# Oracle PL/SQL, Forms, and Reports

Status: unvalidated against a live system. Which dictionary views you can read (USER_, ALL_, DBA_) depends on your privileges, and some views depend on the database version and on licensed options.

## Inventory sources

- Code: packages, procedures, functions, triggers, and views from the data dictionary source and dependency views; wrapped PL/SQL is unreadable there and needs its original source. Forms and Reports modules can be converted to XML with the vendor converters where the installed version provides them (older versions may not).
- Entry points: scheduler jobs and legacy `DBA_JOBS`, database links, external tables, Advanced Queuing queues, materialized views, synonyms and grants, VPD policies, callers of `UTL_FILE`, `UTL_HTTP`, `DBMS_PIPE`, and `DBMS_ALERT`, application server endpoints, Forms menus and libraries (.pll, object libraries).
- Usage: audit trails, `V$SQL` statistics, and table modification tracking (updated lazily). The workload repository (AWR, `DBA_HIST_*`) needs a licensed option and is absent on some editions; Statspack is the unlicensed alternative.

## Hidden behavior to turn into rules or questions

- Business rules living in triggers and Forms triggers; trigger firing order and cascades.
- The empty string equals null; three-valued logic; implicit type conversions and NLS settings (dates, decimal separators, sort order).
- DATE carries a time part; sequences may have gaps; row ordering without an explicit order clause.
- Commits inside procedures, autonomous transactions, and locking behavior with the default isolation level.
- Dynamic SQL and names resolved at run time, which the dependency views cannot see.
- Numeric types: bare NUMBER (floating decimal, up to 38 digits), NUMBER(p) (scale 0, integers), and NUMBER(p,s) map to different target types; `ROUND` rounds half away from zero, which is not the default of every target language.
- CHAR comparison pads with blanks; length semantics (bytes versus characters, `NLS_LENGTH_SEMANTICS`); DDL commits implicitly.
- Forms: implicit record locking, `POST` versus `COMMIT_FORM`, record status, pre-query and post-query triggers, list of values and record groups holding SQL; Reports lexical parameters (`&param`) are dynamic SQL.

## Test-oracle design

Run the same inputs through the packages in a test schema and through the target; compare table states, output files, and messages. Cover null, empty string, time-part, and locale cases explicitly.
