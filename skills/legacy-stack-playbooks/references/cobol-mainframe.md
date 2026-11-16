# COBOL on mainframe: JCL, CICS or IMS, VSAM, DB2

Status: unvalidated against a live system. Check each tool against the site's own scheduler and catalog.

## Inventory sources

- Source libraries and copybooks; compile listings show the copybooks actually used. Load modules without source are flagged.
- JCL procedures and the job scheduler calendar give batch flows, step order, condition codes, and restart points.
- Online: CICS transaction and program definitions, IMS transactions, screen maps.
- Data: VSAM clusters, generation data groups, DB2 catalog, sequential files exchanged with partners.
- Usage: scheduler history, SMF records, transaction statistics, program call counts from the monitor.

## Hidden behavior to turn into rules or questions

- Packed decimal and binary fields, PIC clauses that truncate silently, the ROUNDED phrase, and size-error handling.
- REDEFINES, OCCURS DEPENDING ON, and 88-level conditions that hide code tables.
- Control flow through PERFORM THRU and GO TO; paragraphs shared across code paths.
- File status codes and who checks them; abend handling, checkpoints, and restart logic in batch.
- Sort and merge steps whose control cards define business ordering; EBCDIC collation.
- Generation data groups and date-based dataset names used as implicit parameters.
- Online pseudo-conversational state kept in the communication area or temporary storage.

## Oracle design

Capture input datasets and the matching output datasets per job run, including rejects and return codes; replay through the target batch and compare record by record with the contractual rounding. Test the restart after an injected failure.
