---
name: delphi-memory
description: Diagnose Delphi leaks, double frees, dangling references, constructor failures, interface reference counting, and managed-record ownership.
---

# Ownership, interfaces, and exception safety

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Write an ownership map

For each allocated object or handle, identify who releases it, when ownership transfers, who borrows it, and which callbacks can outlive it. Use the actual API contract: manual `TObject` lifetime, component ownership, an owning collection, interface reference counting, or a custom managed record. Delphi 11+ does not use the old mobile object-ARC model. Managed strings, arrays, interfaces, and anonymous-method references still have compiler-managed lifetimes. [H ch. 7, 11, 13]

A borrowed reference can be non-nil after its object has been freed. `Assigned`, `is`, `ClassName`, and `FreeAndNil` cannot prove arbitrary pointers are valid. `FreeAndNil` only clears the variable passed to it; other aliases remain dangling. Prefer fixing ownership over scattering nil checks.

## Protect acquisitions immediately

For an ordinary object created for local use, place successful construction before its `try/finally` and release it in `finally`. Protect a first acquisition before attempting a second one. This original pattern illustrates the invariant:

```pascal
First := TMemoryStream.Create;
try
  Second := TStringList.Create;
  try
    // Work with both resources.
  finally
    Second.Free;
  end;
finally
  First.Free;
end;
```

It requires `System.Classes` and declared variables. If the second constructor raises, the first stream is still freed. An alternative is to initialize all local references to nil before a combined protected block; do not rely on uninitialized locals being nil. [H ch. 9, 13]

A failing object constructor triggers destruction of the partially constructed instance. Its destructor must tolerate fields whose construction never happened. Calling `.Free` on a nil field is valid; calling `.Clear` first is not. Avoid exceptions escaping destructors/finalizers, especially while another exception is unwinding. Release owned fields before calling `inherited Destroy` according to the ancestor's contract.

Use `try/finally` for cleanup and `try/except` for handling failures. Use bare `raise;` to propagate the current exception. Do not swallow an exception and return apparently successful output. Catch at a boundary where the code can recover, translate to a meaningful domain error, or report failure. Restoring cursor state, locks, temporary settings, and dataset controls also requires exception-safe cleanup. [H ch. 9]

## Interfaces are a lifetime contract

A reference-counted `TInterfacedObject` should normally be held through interfaces throughout its consumer-facing lifetime. Mixing a raw object variable with a temporary interface conversion can destroy the object at the end of a call, leaving the object variable dangling. Do not manually free an object still owned by interface references.

Not every interface implementation is reference-counted: owned components can deliberately disable reference counting. Inspect `_AddRef`/`_Release` or the ancestor contract. A `const` interface parameter is not an extra ownership guarantee. Resolve strong reference cycles explicitly; anonymous methods capturing interfaces can participate in them. Do not add `[weak]` or `[unsafe]` as magic leak fixes without checking supported semantics and the lifetime of the target. [H ch. 11, 13]

## Records and containers

A dynamic array manages its storage, not the lifetime of object references in its elements. An owning object list is a different contract from `TList<TObject>`. Confirm ownership flags before manually freeing elements or extracting them.

Custom managed records can release resources automatically, but copying an owning raw pointer into two records causes double release unless assignment semantics are designed. Review `Initialize`, `Finalize`, and `Assign`, including self-assignment, replacement of an existing destination, temporaries, parameter passing, arrays, and exceptions. Do not apply `FillChar`/`Move` to live managed records. Initialization that raises must clean up acquisitions it made before failing; do not assume its matching finalizer runs. [H ch. 5; official managed-record documentation]

## Verification

Reproduce the normal path and every acquisition-failure path. Test ownership transfer, early exit, removal from an owning container, partial construction, repeated close, and delayed callbacks. Use the project's available leak tooling; on applicable Windows targets `ReportMemoryLeaksOnShutdown` can assist but is not a proof against use-after-free or all native resource leaks. Report allocations/handles separately from merely reachable memory.

## Sources and interpretation

H: *Object Pascal Handbook 11*, ch. 5 pp. 153–156; ch. 9 pp. 255–274; ch. 11 pp. 310–318; ch. 13 pp. 354–379. Key PDF anchors: 268–269, 317–318, 362–363, 376–378. Supplement: [Custom Managed Records](https://docwiki.embarcadero.com/RADStudio/Florence/en/Custom_Managed_Records) and [Using Reference Counting](https://docwiki.embarcadero.com/RADStudio/Florence/en/Using_Reference_Counting). Failure-path tests and the example are original editorial work.
