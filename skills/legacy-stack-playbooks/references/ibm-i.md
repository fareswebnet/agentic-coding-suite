# IBM i (AS/400): RPG, CL, DDS, DB2 for i

Status: unvalidated against a live system. Check that each command or catalog view exists on the installed release.

## Inventory sources

- Source members: source physical files (RPG, CL, DDS, display and printer files) per library; also compiled objects without source, which need decompilation or runtime tracing and must be flagged. Source often differs from the compiled object (edited after the last compile, or compiled from another library): compare compile timestamps and the source information recorded in the program before trusting the source.
- Object catalog: `DSPOBJD` with full detail, or SQL services such as `QSYS2.OBJECT_STATISTICS` (7.2 or later; also look at the program, module, routine, trigger, and scheduled-job information services and `GENERATE_SQL` for DDL, and confirm their names on your release), give type, owner, creation date, and usage data. Usage data is weak evidence: the time part of the last-used timestamp is zero, the count is valid only since the last reset (record the reset date), an empty value means not used since the reset rather than never used, and some object types do not update it. Prefer real history where it exists: the audit journal if enabled, job logs, and collection-services data.
- References: `DSPPGMREF` lists static, compile-time references only; it misses calls whose program name is a variable, commands built as strings and run through `QCMDEXC` or `SBMJOB`, and prepared dynamic SQL, so add a source search and runtime tracing for those; `DSPDBR` lists logical files and dependencies over a physical file; `DSPFFD` and `DSPFD` give fields and file attributes.
- Entry points: menus and command objects, job schedule entries and the job scheduler, subsystem and autostart job descriptions, data queues and message queues, output queues, inbound and outbound FTP and file transfers, SQL stored procedures and triggers, and exit programs.
- Hidden inputs: data areas (including the local data area), user spaces, `QTEMP` content, keyed data queues, library list order, override commands, job descriptions, environment variables, and system values (for example the default CCSID, date format and separator, century, startup program, and user library list). A CCSID of 65535 on a file or job disables conversion silently.

## Hidden behavior to turn into rules or questions

- Numeric types: packed and zoned decimals, implied decimal positions, truncation on overflow, and rounding: truncation is the default and half-adjust is an explicit operation extender or keyword, so check each calculation; invalid decimal data (a runtime error such as MCH1202) can stop a program that tolerated it on another path.
- Indicators as hidden state and as control flow; the RPG cycle (implicit reads, total calculations, last-record handling) in older programs.
- Dates and times: numeric dates, zero or low and high values standing for "no date", and century handling.
- Text: fixed-length blank-padded fields, EBCDIC collation, CCSID conversions, and sort order differences.
- Data: null versus default value, record-level locking, whether files are journaled and commitment control is used (no journal means no rollback).
- Same program name in several libraries; behavior depending on the library list.
- CL error handling through message monitoring and job log messages; output through spooled files, which are interfaces other systems may read.
- Adopted authority (programs running with the owner's rights) and shared service profiles: record them in the security baseline.

## Test-oracle design

Run the legacy program on a test partition with captured inputs and compare database files, data areas, queues, spooled output, and messages. Include month-end and error paths.
