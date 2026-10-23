---
name: delphi-testing
description: Write meaningful Delphi regression tests, use DUnit or DUnitX correctly, debug runtime failures, and assess performance with measurements.
---

# Testing, debugging, and measured performance

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Test the observable contract

Use the project's existing test framework and runner. Prefer focused behavior tests around changed logic; do not replace an established DUnit suite with DUnitX merely for modernity. Separate deterministic unit tests from database/network/GUI integration tests and device/deployment tests. A passing algorithm test is not proof that its form resource or driver loads. [E ch. 14; M ch. 18]

For a bug fix, make the test fail for the original bug and pass with the change when practical. Assert returned data, state transitions, ownership/lifetime events, or defined errors. Avoid tests that repeat implementation line for line or merely search source text for a preferred idiom.

## DUnitX lifecycle correction

In DUnitX, `[Setup]` and `[TearDown]` apply around **each test**; `[SetupFixture]` and `[TearDownFixture]` apply to the fixture lifecycle. *Expert Delphi* p. 382 loosely describes setup/teardown as occurring before/after all tests; use the framework's distinct attributes instead of that wording. Inspect the version bundled with the target project, its RTTI requirements, fixture registration, and runner options. [DUnitX primary examples and maintainer clarification]

Keep tests independent. Reset owned resources and state for each case unless sharing is explicit and safe. Verify that the runner actually discovers the expected tests: “zero tests passed” is not successful coverage. CI needs a failing exit status and retained diagnostics when tests fail; do not rely solely on a green-colored console message.

## Delphi-specific regression cases

- Ownership: exception on a later allocation, partial constructor failure, owning-list removal, interface/object mixing, and a callback after owner destruction.
- Data: empty arrays, duplicate/missing dictionary keys, null versus empty fields, invalid JSON root/type, transaction rollback, and stale persistent fields.
- Text: UTF-8/UTF-16 conversion, supplementary characters, locale-dependent formats, and ANSI boundaries.
- UI/resources: actual construction of changed forms/frames, event bindings, close/reopen behavior, DPI/layout, and invalid style resources.
- Concurrency: cancellation before/after completion, repeated start/stop, pending queue at shutdown, and worker exceptions.
- Portability: minimum compiler, active conditionals, pointer-width boundary values, and intended library bitness.

Choose cases relevant to the change rather than creating every test for every task.

## Debug from the first failure

Reproduce with a known input and configuration. Capture the first meaningful exception, call stack, and object state before later cleanup obscures it. Breakpoints, watches, and conditional breakpoints should narrow a hypothesis. Avoid evaluating property getters or functions in a watch if their side effects can change behavior. Optimized code may not map straightforwardly to source or retain all locals. [M ch. 18]

Use range/overflow checking and appropriate debug information in a diagnostic configuration where useful. Do not silently change release compiler settings to conceal an error. Memory leak reporting is one tool; inspect use-after-free, resource handles, thread shutdown, and module lifetime separately. Keep error reproduction and logs free of sensitive production data.

## Measure before optimizing

Use `TStopwatch` or the existing profiler for elapsed-time measurements. Distinguish timer resolution from accuracy and benchmark reproducibility; a high-frequency counter does not establish a precise single-run performance result. Warm up relevant caches, repeat representative workloads, and keep compiler/target/configuration comparable. [F pp. 84–99; E ch. 4]

Measure the suspected bottleneck separately from UI updates, logging, data generation, and startup. Consider algorithm choice, I/O round trips, allocations, and contention before low-level rewrites. Keep a sequential baseline when adding parallel work. Check that outputs remain equivalent and that gains survive realistic data sizes; small tasks may lose to scheduling overhead.

## Completion evidence

Report the actual runner and test count/result, compiler/configuration, scenarios exercised, and relevant benchmark conditions. If no Delphi toolchain is present, mark code examples and proposed tests as uncompiled/unrun. Static review can identify risks but cannot substitute for runtime evidence.

## Sources and interpretation

E: *Expert Delphi*, ch. 14 pp. 379–382 (PDF 402–405), ch. 4 pp. 91–103. F: *Code Faster in Delphi*, pp. 84–99 (PDF 87–102). M: *Mastering Delphi 5, 2025 Annotated Edition*, ch. 18 PDF 833–867. H: *Object Pascal Handbook 11*, ch. 13 pp. 368–379. Correction sources: [DUnitX examples](https://github.com/VSoftTechnologies/DUnitX/blob/master/Examples/DUnitX.Examples.General.pas), [maintainer clarification of fixture setup](https://github.com/VSoftTechnologies/DUnitX/issues/311). Regression selection, benchmark controls, and evidence reporting are editorial synthesis.
