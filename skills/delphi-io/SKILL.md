---
name: delphi-io
description: Implement Delphi file and stream I/O, JSON or XML parsing, serialization ownership, encoding, and schema validation.
---

# Streams, files, JSON, and XML

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Define the boundary contract

Identify encoding, maximum input size, expected shape, missing/null behavior, numeric/date representation, and who owns each stream or tree node. Choose DOM parsing when convenient random access matters; choose streaming for large sequential inputs. A streaming parser still needs state and nesting validation; recognizing any matching property name anywhere is not a schema check. [E ch. 3; H ch. 18]

Use `System.IOUtils` paths and file APIs where appropriate. Construct paths with `TPath.Combine`; decide whether the path is writable on the target. If an external name contributes to a path, ensure it cannot escape the intended directory. Define overwrite and partial-write behavior. For important persistent state, write a complete temporary file and replace the destination using a supported strategy rather than leaving a half-written file on failure. This is an editorial reliability extension to the books' basic file examples.

## Streams and wrappers

Streams carry bytes, a position, and capabilities. Do not assume every stream can seek or exposes a meaningful size. After writing into a memory stream, set its position deliberately before reading from it. Check exact versus partial-read APIs: use a buffer operation that raises on a short read when the format requires an exact number of bytes. Distinguish byte counts from character counts. [H pp. 509–512; E pp. 67–73]

Protect streams immediately after successful creation. A reader/writer may own its underlying stream depending on the constructor/option used. Make the ownership choice explicit, flush buffered output at the required point, and release wrappers before the resources they borrow. Do not double-free a stream already owned by its wrapper. Keep asynchronous consumers from outliving a local stream.

Use explicit text encoding for persisted or network text. Inspect BOM expectations and avoid relying on a default encoding changing between APIs. A successful file-open is not evidence the contents decode correctly.

## JSON trees

Treat a parsed root as an owned resource, and children retrieved from it as borrowed unless the API says otherwise. Free the root in `finally`. Do not free children still owned by the tree. When adding a separately allocated child, understand whether and when ownership transfers, including failure. [E pp. 73–83; memory skill]

Inspect the chosen `ParseJSONValue` overload and options: malformed input may produce nil or an exception depending on the call. Check the root type before casting, then check required fields, element kinds, ranges, and nullability. Distinguish an absent field, JSON null, empty string, and zero. A coercion that yields a default is not always valid input.

Use the JSON library for escaping; do not build JSON by concatenating user strings. Define duplicate-property policy and preserve identifiers that cannot safely be represented by a consumer's floating-point number. Keep model validation separate from transport parsing.

The JSON Data Binding Wizard described in *Expert Delphi* is a **Delphi 12+ IDE feature**, not a prerequisite for Delphi 11. Generated types or runtime mapping may still be usable if their actual dependencies compile on the baseline; inspect them rather than rejecting or accepting them based only on how they were generated.

## XML

Choose an XML implementation available on each target. Check namespace URIs, not merely prefixes; validate required nodes and attribute conversions. DOM interfaces and COM-backed implementations have different lifetime and platform implications from raw object trees. Inspect parser options for external resource resolution when accepting external XML. Do not assume Windows-specific XML vendors exist on mobile. [E pp. 85–90]

## Verification

Test empty and truncated files, malformed JSON/XML, wrong root type, missing/null fields, unknown fields, invalid numeric ranges, Unicode, duplicate keys where relevant, read-only paths, and exceptions during output. Include a round-trip test for every persisted format change and compatibility with a previously saved file. Verify tree and stream cleanup on failure.

## Sources and interpretation

E: *Expert Delphi*, ch. 3 pp. 67–90 (PDF 90–113), especially PDF 103–107 for JSON and the Delphi 12 wizard. H: *Object Pascal Handbook 11*, ch. 6 pp. 195–199 and ch. 18 pp. 507–515. Supplement: [What's New in Athens](https://docwiki.embarcadero.com/RADStudio/Athens/en/What%27s_New). Input validation, atomic-save strategy, and adverse-input tests are editorial extensions; examples in a book are not complete production parsing contracts.
