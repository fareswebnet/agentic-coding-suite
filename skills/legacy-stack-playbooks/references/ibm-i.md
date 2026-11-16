# IBM i (AS/400): RPG, CL, DDS, DB2 for i

Status: unvalidated against a live system. Check that each command or catalog view exists on the installed release.

## Inventory sources

- Source members: source physical files (RPG, CL, DDS, display and printer files) per library; also compiled objects without source, which need decompilation or runtime tracing and must be flagged.
- Object catalog: `DSPOBJD` with full detail, or the SQL catalog services, give type, owner, creation date, and usage data such as last-used date and days-used count; reset dates matter, record them.
- References: `DSPPGMREF` lists the files and programs each program uses; `DSPDBR` lists logical files and dependencies over a physical file; `DSPFFD` and `DSPFD` give fields and file attributes.
- Entry points: menus and command objects, job schedule entries and the job scheduler, subsystem and autostart job descriptions, data queues and message queues, output queues, inbound and outbound FTP and file transfers, SQL stored procedures and triggers, and exit programs.
- Hidden inputs: data areas (including the local data area), library list order, override commands, job descriptions, and environment variables.

## Hidden behavior to turn into rules or questions

- Numeric types: packed and zoned decimals, implied decimal positions, truncation on overflow, and the half-adjust rounding operation versus plain truncation.
- Indicators as hidden state and as control flow; the RPG cycle (implicit reads, total calls, last-record handling) in older programs.
- Dates and times: numeric dates, zero or low and high values standing for "no date", and century handling.
- Text: fixed-length blank-padded fields, EBCDIC collation, CCSID conversions, and sort order differences.
- Data: null versus default value, record-level locking, whether files are journaled and commitment control is used (no journal means no rollback).
- Same program name in several libraries; behavior depending on the library list.
- CL error handling through message monitoring and job log messages; output through spooled files, which are interfaces other systems may read.
- Adopted authority (programs running with the owner's rights) and shared service profiles: record them in the security baseline.

## Oracle design

Run the legacy program on a test partition with captured inputs and compare database files, data areas, queues, spooled output, and messages. Include month-end and error paths.
