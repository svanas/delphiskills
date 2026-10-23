---
name: delphi-concurrency
description: Implement or review Delphi TThread, PPL tasks and futures, synchronization, cancellation, anonymous-method capture, and main-thread callbacks.
---

# Threads, tasks, closures, and UI delivery

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Start with lifetime and completion

Before adding a task, identify the inputs it owns, data it shares, cancellation signal, result destination, exception observer, and shutdown path. An interface to an `ITask` is not proof that a form captured by its closure remains alive. Ordinary object references captured by anonymous methods are not automatically owning references on Delphi 11+. [H ch. 15; E ch. 4; F “Variable Capture”]

Snapshot input on the UI thread before dispatching. Capture stable values or owned state, not a live edit control or a dataset cursor being navigated elsewhere. Closures capture variables, not just their value at creation. Avoid a shared loop variable accidentally feeding every queued operation; pass a value through a separate routine invocation or another clearly independent per-operation scope.

## Select the primitive

- Use `TThread` for a long-lived worker with explicit startup/shutdown or thread-affine resources. Initialize fields before starting it. Prefer creation suspended followed by `Start` to racing construction. `Terminate` requests cooperation; it does not forcibly stop blocking work.
- Use `TTask.Run` for work to schedule immediately, or create a task and explicitly start it when delayed scheduling is intended. Do not call `Start` twice.
- Use `TParallel.For` for independent work with sufficient granularity. It waits for completion; placing it directly in a UI event does not make that event asynchronous.
- A future's `.Value` can block. Reading it immediately on the UI thread can recreate the freeze that background computation was meant to avoid.
- Do not add parallelism to compensate for avoidable database/network round trips before measuring them. [E ch. 4; F printed pp. 90–99]

## Synchronize invariants, not isolated lines

Use a lock for a compound invariant such as checking a dictionary and then inserting. `TInterlocked` fits supported atomic scalar operations, not arbitrary object graph consistency. All readers and writers of the protected state must follow the same protocol. Release acquired locks in `finally`; keep critical sections short and establish a consistent lock order. Do not call user callbacks, wait on workers, or marshal to the UI while holding a lock needed by the recipient.

`System.TMonitor` is distinct from similarly named UI types. Its lock target must remain alive throughout use. Wait/pulse protocols require a condition checked in a loop; a pulse is not persistent completion state. [E printed pp. 96–98]

## Deliver UI work safely

VCL and FMX UI access belongs on the main thread. `Synchronize` waits for main-thread execution; `Queue` normally schedules and lets the worker continue. **When called from the main thread, `Queue` can execute immediately**; use `ForceQueue` if deferred execution is required. Do not infer timing solely from the method name. [Official Queue documentation]

Plan what happens if the form closes after the callback is queued. A safe design can use a controller whose lifetime spans all work, an explicit invalidation/cancellation protocol checked before delivery, or a join-and-drain arrangement that cannot deadlock. A nil check on a captured stale object is insufficient. Associate queued callbacks with a removable owner/thread when using supported removal APIs; `Queue(nil, ...)` does not by itself provide a per-worker cancellation handle. Remove or invalidate pending work before destroying its target.

Avoid a shutdown cycle in which the main thread waits for a worker while the worker synchronously waits for the main thread. Do not assume every wait pumps messages correctly on every target. `Application.ProcessMessages` adds reentrancy; it is not a replacement for a proper asynchronous design.

## Errors, cancellation, and tests

Define how task exceptions reach the operation's result and UI. A global form exception handler does not establish observation of every background failure. Catch only when converting to a defined error result or reporting; release resources on all paths. Cancellation needs cooperative checkpoints and a way to unblock waiting I/O where applicable.

Test close-during-work, repeated start/cancel, completion after cancellation, queue delivery after target destruction, worker exceptions, empty work, and shutdown with pending callbacks. Run stress repetitions and use timeouts to detect deadlocks; replace sleep-based ordering with synchronization in tests. A speedup claim must include workload, target, and measured results.

## Sources and interpretation

H: *Object Pascal Handbook 11*, ch. 15 pp. 430–448. E: Cantù and Głowacki, *Expert Delphi*, second edition (2024), ch. 4 pp. 91–103 (PDF 114–126). F: *Code Faster in Delphi*, pp. 77–83 and 90–99 (PDF +3). M: *Mastering Delphi 5, 2025 Annotated Edition*, ch. 17, PDF p. 804 footnote 364 corrects the historical background-work emphasis. Supplement: [TThread.Queue, Alexandria](https://docwiki.embarcadero.com/Libraries/Alexandria/en/System.Classes.TThread.Queue). Shutdown/cancellation protocols are editorial extensions to the books' introductory examples.
