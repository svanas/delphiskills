---
name: delphi-language
description: Review and write Delphi units, types, routines, classes, records, properties, and compiler conditionals for Delphi 11 or newer.
---

# Object Pascal language and API design

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Establish the contract before changing syntax

Read the unit's interface and its callers before changing its implementation. Public signatures, default arguments, overload resolution, calling conventions, exception behavior, and serialized names are compatibility surfaces. Keep implementation-only dependencies in the implementation `uses` clause when possible; moving a dependency can change initialization order, so inspect initialization/finalization sections as well as circular references. Use qualified unit/type names when ambiguity is real; do not rewrite all uses clauses merely for style. [H ch. 1, 4]

In reviews, distinguish a real language error from a local convention. Pascal identifiers are case-insensitive, but filenames, deployment destinations, and external identifiers may not be. Preserve existing indentation and naming. Avoid introducing `with` where implicit member resolution makes a later class change alter meaning. Unqualified `private` and `protected` have unit-scope accessibility rules; choose `strict private` or `strict protected` when actual class-level restriction is required. [H ch. 5, 7]

## Variables, parameters, and results

- Initialize ordinary local scalars, object references, and plain records before use. Compiler-managed fields receive special treatment; that does not initialize every field of a record. Instance fields start zeroed, but a constructor can fail partway through initialization.
- Inline variables are available at the Delphi 11 baseline. Use them to narrow scope when consistent with the codebase; a block ending can finalize a managed value earlier than a routine-level declaration would.
- Use `const` for read-only parameters when its semantics fit. It neither clones a referenced object nor makes its internal state immutable. `var` permits replacement of the caller's value. `out` communicates output-only intent; initialize the output on every successful return, and do not read its old value.
- Do not change an interface parameter from value to `const` as a mechanical optimization: the lifetime protection supplied by a reference-counted temporary can change.
- Assign `Result` on all normal paths. Preserve the distinction between a Boolean `Try...` API for expected absence and an exception for an operation that cannot fulfill its contract.
- Make width and range intentional. `Integer` remains a 32-bit type on Win64; `NativeInt`/`NativeUInt` follow pointer size. Use fixed-width protocol fields and pointer-sized types for pointer arithmetic, not one type for every number. [H ch. 2, 4, 13]

## Classes, records, dispatch

Choose a record for value semantics and a class for shared identity and explicit lifetime. Assignment of a class variable copies a reference; record assignment copies fields, with managed-field semantics and any custom `Assign` operator. A record containing an object reference does not thereby own or clone that object.

Use `override` to participate in an inherited virtual/dynamic dispatch slot. `reintroduce` suppresses a hiding warning; it does not override that slot. Adding `virtual` to a descendant method cannot retroactively change base-class calls. Evaluate constructor dispatch when using `class of` references. Destructors normally override `Destroy` and call the inherited destructor. [H ch. 7–8, 12]

Keep property getters predictable and setters responsible for invariants. Return values from record-valued properties are not necessarily mutable references to the stored record: read, modify, and assign back when needed. A `default` property directive describes persistence metadata; it does not initialize a field. Events declared `procedure ... of object` contain a code pointer and an instance pointer; they are not anonymous-method reference types. [H ch. 10]

## Conditionals and expressions

Inspect active include files and `.dproj` defines before reasoning about a branch. Use `{$IF CompilerVersion >= ...}` for a version threshold and target symbols for platform differences; keep those two questions separate. A later compiler number does not imply a library or platform feature exists. Avoid placing newer syntax in shared code without a supported conditional alternative.

Check Boolean evaluation mode before relying on a guard such as `(Obj <> nil) and Obj.Ready`. Prefer an explicit nested check where changed directives or evaluation order would make correctness fragile. Parenthesize nontrivial mixed Boolean and bitwise expressions. Preserve `div` versus `/`, signedness, overflow behavior, enum values, and set ranges during refactoring. Never renumber persisted enum ordinals casually. [H ch. 1–3]

## Verification

Compile changed public interfaces against representative callers. Exercise nil/empty input, boundary values, every overload used by clients, and base-class dispatch to descendants. For a record change, test assignment and passing by value. For conditional code, build each affected branch rather than counting a successful build of one configuration as coverage of all branches.

## Sources and interpretation

H: Marco Cantù, *Object Pascal Handbook, Delphi 11 Alexandria Edition* (2021), ch. 1–5, 7–8, 10, 12–13. Specific anchors: printed pp. 38–47 (units/directives), 53–59 (initialization/scope), 106–119 (parameters), 137–156 (records), 210–214 (visibility), 239–244 (dispatch), 275–295 (properties/events). PDF page = printed page + 1. The API-preservation and verification procedure is editorial synthesis.
