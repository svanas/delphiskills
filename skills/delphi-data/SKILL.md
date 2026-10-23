---
name: delphi-data
description: Review Delphi FireDAC connections, parameterized queries, dataset state, transactions, threading, bindings, and BDE migration.
---

# FireDAC, datasets, and database migration

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Identify the actual data layer

Read the connection configuration, queries, data modules, persistent fields, bindings, and schema together. Prefer the established supported data layer; use FireDAC for new work where its drivers and project requirements fit. Do not introduce BDE, Paradox configuration, or dbExpress merely because the Delphi 5 text describes them. Existing ADO/dbGo or IBX code is not automatically wrong and need not be replaced for an unrelated change. [M annotations to ch. 9; E ch. 10; F “BDE Replacement”]

Keep connection details configurable and credentials out of source, DFM/FMX resources, and diagnostic logs. Set design-time connections inactive for deployable resources unless the application explicitly requires and safely handles startup connection behavior. Verify driver availability, client library bitness, target support, and deployment prerequisites.

## Query semantics

Bind values as parameters with intentional types, sizes, and null handling. SQL identifiers cannot generally be replaced by value parameters; map dynamic table/order choices through a controlled set. Use `Open` for result sets and `ExecSQL` for commands according to the query's contract. Verify affected-row expectations for updates and deletes.

Do not infer null from a field's default converted value. Inspect `IsNull` and preserve the domain distinction between null, zero, and empty text. Persistent fields can become stale after schema changes: verify type, size, precision, required flags, and field names. Use the database's appropriate decimal/money representation rather than assuming binary floating point is exact for amounts. [M ch. 9–11; E ch. 10]

## Dataset state and bindings

Understand `dsBrowse`, edit/insert state, `Post`, and `Cancel`. Posting a row to a dataset is not necessarily committing the database transaction, especially with cached updates. Inspect `CachedUpdates`, update application, reconciliation, and transaction settings before promising durability.

Wrap `DisableControls`/`EnableControls` in `try/finally` during bulk iteration. Preserve and restore position only if the dataset supports the chosen bookmark strategy and the bookmarked row remains valid. Do not mutate a bound dataset from a worker while the UI reads it. Data modules organize data access but do not by themselves create a thread-safety boundary. [M ch. 9–10; E ch. 10]

## Transactions

Decide whether the routine owns a new transaction or participates in its caller's transaction. Do not blindly commit or roll back a transaction started by another layer. For a unit of work, start, execute related changes, commit on success, and roll back the owned transaction on failure while preserving useful error context.

FireDAC's auto-commit does not make multiple independent statements one atomic operation. Nested transaction behavior depends on options and database savepoint support. A commit/rollback can invalidate open cursors; verify the database and fetch settings. `CommitRetaining` does not mean the transaction has ended. [Official FireDAC transaction documentation]

Define an explicit concurrency policy for edits: an update by key alone may overwrite another user's changes. Use a version/original-value predicate or the project's equivalent and report conflict distinctly from “row missing.” Keep database transactions short; avoid holding one open while waiting for user interaction or an external HTTP service.

## Threads and connection pooling

Use a dedicated connection and its associated query/transaction objects per worker as the ordinary pattern. Activate `FDManager` before workers start. A connection and its related objects may only be used by one thread at a time; locking a single query call is insufficient if another thread accesses the open cursor or active transaction. Pooling can reuse physical connections; it does not permit simultaneous use of one connection object. [Official FireDAC multithreading documentation]

## Migration and verification

BDE migration must reconcile units, component classes, connection properties, field definitions, parameter behavior, SQL dialect, transactions, sorting/collation, and error handling. Convert PAS and DFM/FMX together and test migrated forms at runtime. Use repeatable scoped transformations; a class-name replacement is not a semantic migration. [F pp. 136–145]

Test rollback after the second of several writes fails, null versus empty values, duplicate keys, concurrent edits, schema upgrades, connection failure, and a clean deployment. Use disposable test data or the project's test database. Do not use a production database as a convenient integration fixture.

## Sources and interpretation

E: *Expert Delphi*, ch. 10 pp. 265–297 (PDF 288–320). F: *Code Faster in Delphi*, pp. 108–113 and 136–145 (PDF +3). M: *Mastering Delphi 5, 2025 Annotated Edition*, PDF 401 footnote 210, PDF 403 footnotes 213–217; enduring dataset concepts in ch. 9–11. Supplements: [Managing Transactions](https://docwiki.embarcadero.com/RADStudio/Florence/en/Managing_Transactions_%28FireDAC%29), [Multithreading, Alexandria](https://docwiki.embarcadero.com/RADStudio/Alexandria/en/Multithreading_%28FireDAC%29). Conflict handling, credential handling, and verification are editorial operational guidance.
