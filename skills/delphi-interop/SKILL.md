---
name: delphi-interop
description: Review Delphi native API declarations, pointer widths, DLL interfaces, binary compatibility, runtime packages, COM, and Automation.
---

# Windows APIs, DLLs, packages, and COM

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Match the foreign contract exactly

Read the actual API declaration/header or imported type library before writing a Delphi declaration. Match parameter width, signedness, Boolean representation, alignment, calling convention, character encoding, and ownership. A cast that makes the compiler accept a call is not evidence of ABI compatibility. [H ch. 4–5; M ch. 14–16]

Use platform declarations such as handles and pointer-sized integer types where appropriate; do not cast a pointer to `Integer` on Win64. Conversely, a protocol's fixed 32-bit integer should not grow just because the host is 64-bit. Check record packing and `SizeOf` against the external structure rather than applying `packed` everywhere. Include version/size fields when required by the API.

The caller and a directly loaded native library must have matching bitness. This includes database client DLLs and IDE-loaded packages; target application architecture and IDE architecture are separate. [M PDF 675 annotation]

## Exported boundaries

Use a deliberate stable boundary for libraries consumed by other languages: documented scalar/buffer types or an established COM contract. Do not casually export Delphi classes, UnicodeString, dynamic arrays, or exceptions through an arbitrary C ABI. Document allocation/freeing responsibility and provide a matching release routine when a library allocates a returned buffer. Sharing managed Delphi data across modules requires a compatible runtime/memory-management arrangement, not just identical declarations.

Catch exceptions at a foreign boundary and translate to the agreed status/error contract. Choose the actual ABI's calling convention; the Delphi 5 recommendation of `stdcall` reflects Win32 conventions, not a universal rule for every target/API. Keep callback code and its instance/context alive while foreign code can call it.

Load only the intended library and check load/symbol failures. Do not unload while callbacks, exported object instances, worker threads, or cached function pointers still depend on the module. Clear borrowed references after unloading. [M ch. 14; editorial lifecycle guidance]

## Runtime packages

Use compatible compiler/target/package builds for BPL/DCP/DCU artifacts. Separate outputs by compiler, configuration, and platform. A stale DCU can produce misleading build errors or conceal the source version actually compiled. Review package `requires` dependencies and avoid compiling the same unit independently into conflicting runtime modules.

Packages can preserve Delphi type integration within a compatible ecosystem; they are not a stable cross-version plugin ABI. Keep design-time dependencies out of runtime deployments and rebuild dependent packages when compatibility requires it.

## COM and Automation

COM interfaces are contracts identified by IIDs; do not change a published interface layout in place and assume old clients remain binary-compatible. Distinguish a local Delphi interface abstraction from a registered COM server/type-library contract. Use imported declarations when appropriate and preserve generated versus handwritten code boundaries. [M ch. 15–16; H ch. 11]

For each thread making COM calls, verify its COM initialization and apartment requirements. Balance successful initialization with corresponding uninitialization and use marshaling when crossing apartments; copying an interface reference does not automatically make an apartment-bound object thread-safe. Prefer early-bound typed interfaces when the contract is available, and use late-bound Automation deliberately with explicit variant/error handling. Check registration and bitness on a clean machine. These thread/apartment checks are editorial additions that must be confirmed against the API used.

## Verification

Build and run the caller/library combinations actually supported. Check record sizes, buffer lengths, Unicode/ANSI boundaries, missing libraries, missing exports, error translation, callback shutdown, and object release before unloading. For COM, exercise activation and calls from the intended thread/apartment. Do not use a successful Win32 test as evidence for Win64 interoperability.

## Sources and interpretation

H: *Object Pascal Handbook 11*, ch. 4 pp. 119–124, ch. 5 pp. 137–164, ch. 11 pp. 310–329. M: *Mastering Delphi 5, 2025 Annotated Edition*, ch. 14 PDF 668–710 (especially 675 and 681–688), ch. 15 PDF 711–744, ch. 16 PDF 745–798. PDF 711 footnote 355 explains COM's continued relevance. ABI review and modern thread/lifetime checks are editorial synthesis, requiring the exact foreign API documentation for implementation.
