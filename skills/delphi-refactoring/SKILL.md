---
name: delphi-refactoring
description: Review Delphi changes for concrete defects and refactor PAS, DFM/FMX, and project files while preserving behavior and version compatibility.
---

# Review, refactoring, and legacy modernization

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Review the behavior that changes

Read the requested change, surrounding implementation, callers, resource files, and affected conditional branches. Trace actual object ownership, data flow, and failure paths. Report a finding only when you can describe the trigger, consequence, location, and a useful correction. Separate correctness defects from optional style preferences. Prioritize a reproducible crash, corruption, leak, or broken contract above formatting. [Editorial review workflow grounded in H ch. 9, 13 and M ch. 18]

Use the relevant topic skills rather than applying a universal checklist to every line. Key Delphi review questions include: who frees this value; which thread calls this event; which compiler branch is active; can this DFM still load; does a renamed member have a persisted or external name; and which code runs if acquisition or parsing fails?

## Preserve contracts during extraction and renaming

Before extracting a method, identify early exits, exceptions, captured variables, inherited calls, and lifetime changes caused by moving inline managed variables. Preserve parameter modes and overload selection. A local helper that captures outer state is not equivalent to a pure function with a copied parameter.

Before moving code between units, inspect interface dependencies, initialization/finalization, class constructors, private/protected unit visibility, and name collisions. Moving code out of a unit can remove access it previously had even when the class visibility label did not change. [H ch. 1, 4, 7, 12, 15]

Before renaming, search Pascal declarations/usages, DFM/FMX event names, component names, project entries, RTTI lookups, serialization mappings, exported names, and generated bindings. Use semantic tools where available. Text search is useful discovery, not proof of a complete semantic rename.

## Transform source and resources together

For a component migration, build an explicit mapping of class, unit, property, event, and behavior changes. Parse DFM/FMX when structural edits matter; a global property deletion can affect unrelated component classes. Preserve property values that have meaningful equivalents and flag changes that require a design decision. [F pp. 136–145]

Scope transformations to the affected files; keep a clean baseline and inspect the diff. Test a representative form/component family before scaling to the entire application. Make the migration repeatable and verify a second run is harmless where the tool is intended to be idempotent. Do not assume the particular plugin or parser mentioned in a 2020 book is installed or supports the project's current syntax.

## Modernize by risk

1. Establish a baseline build and representative behavior tests, or explicitly record why unavailable.
2. Separate a mechanical compatibility change from a behavioral redesign so failures can be localized.
3. Address Unicode and pointer-width assumptions, ownership, and resource compatibility at their boundaries.
4. Replace obsolete dependencies only within the requested scope. A working ADO application does not need an unsolicited FireDAC rewrite.
5. For database migration, verify transactions, field types, null handling, sorting, and error behavior, not just component names.
6. Build affected configurations and run tests after a meaningful batch; inspect the final diff for accidental generated or resource changes.

Historical annotations are evidence, not a promise that all unannotated Delphi 5 statements remain current. Do not revive browser ActiveForms, BDE deployment, CORBA, or old mobile ARC assumptions as new recommendations. Keep COM, VCL, events, datasets, and streaming concepts where they remain applicable. [M preface and annotations; F migration case study]

## Productivity without unnecessary machinery

Use project navigation, code completion, class completion, templates, and existing formatting tools where available. Review generated code and designer edits. Do not automate mouse shortcuts when a direct source edit is clearer, or install IDE extensions merely to follow a book. The transferable lesson from *Code Faster* is to use repeatable, scoped transformations and known RTL functionality; its keyboard maps and plugin versions are environment-specific.

## Verification and output

For reviews, give concrete findings with file/line evidence and state verification limits. If no defect is found, say so without implying exhaustive proof. For refactoring, summarize the preserved contract, changed behavior if any, actual tests/builds, and unresolved compatibility assumptions. Do not mix an unrequested style sweep into the fix.

## Sources and interpretation

F: *Code Faster in Delphi*, pp. 16–41, 51–57, 136–148 (PDF +3), particularly the BDE case study p. 136 and DFM/reFind discussion pp. 140–145. H: *Object Pascal Handbook 11*, ch. 1, 4, 7, 9, 12–15. M: *Mastering Delphi 5, 2025 Annotated Edition*, PDF 5–6 (annotation policy), 401/403 (data access), 943 (ActiveForms), 1007 (CORBA). The review prioritization and migration staging are original operational synthesis.
