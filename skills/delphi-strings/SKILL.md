---
name: delphi-strings
description: Handle Delphi Unicode strings, encodings, indexing, text formatting, and legacy ANSI or pointer-based text interfaces.
---

# Unicode, strings, and binary boundaries

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Identify the representation at each boundary

For every file, database column, HTTP body, DLL parameter, and UI field, distinguish text from bytes. In ordinary modern Delphi code, `string` is UnicodeString, `Char` is a UTF-16 code unit, and `PChar` follows the wide-character representation. `AnsiString`, `UTF8String`, `RawByteString`, and `TBytes` are not interchangeable. Inspect short-string directives and explicitly declared legacy types before generalizing. [H ch. 6]

Keep text as `string` internally and encode/decode at defined boundaries. Use `TEncoding.UTF8` when UTF-8 is the required external contract. Do not select the machine ANSI code page just because a legacy overload accepts it. `RawByteString` has specialized code-page-preserving parameter uses; it is not a universal UTF-8 or binary container. Binary data belongs in bytes or a stream.

## Index and count deliberately

Delphi 11's normal string indexing is one-based; `Chars[]` and many string-helper operations are zero-based. Dynamic arrays are zero-based. Inspect `{$ZEROBASEDSTRINGS}` if present and verify the specific RTL API's index convention instead of translating every index together. `Length(S)` counts UTF-16 code units, not user-visible characters. A supplementary character needs a surrogate pair; a visible grapheme can contain multiple code points. Do not cut arbitrary code units for a UI character limit. [H printed pp. 165–185]

An empty string has no first element. Guard `S[1]`, buffer addresses, and copying operations accordingly. For low-level work use documented lengths rather than searching for a terminating zero unless the foreign API requires zero termination. Delphi strings can contain embedded nulls; C-style consumers may stop early.

## Ownership, aliases, and buffers

UnicodeString assignment shares storage with copy-on-write semantics for normal managed writes. A raw pointer write bypasses those safeguards. A `PChar` obtained from a temporary conversion or concatenation must not escape that temporary's lifetime. Keep a named backing value alive for the full call, and do not resize or mutate it while a foreign consumer retains its pointer.

For an output buffer, establish writable, uniquely owned storage, capacity, length units, and the API's terminator contract. After the call, resize to the actual text length. `SetLength` expands capacity but is not a promise to fill new string characters with spaces or zeros. Do not use pointer-size casts to infer a string's hidden header; that representation is not a stable wire format. [H printed pp. 178–195]

## Formatting and comparison

Use explicit format settings for persistent or network numeric/date representations; retain locale-aware formatting for presentation. Do not serialize a date using the user's display format and then parse it as a protocol date. Specify timezone semantics separately from `TDateTime`, which alone does not encode them.

Choose equality to match the domain: case-sensitive identifiers, a documented case-insensitive key policy, or linguistic display sorting. Avoid applying `LowerCase` to every identifier as a substitute for a defined comparison policy. When moving from `TStringList` to a dictionary, preserve case sensitivity, duplicate handling, sorting, and key normalization explicitly. [H ch. 6, 14, 18]

## Review and test

Look for implicit ANSI conversions, `SizeOf(Char)` assumptions, casts from text to binary, `Length` used as byte count, mismatched `Copy`/helper indexes, and a returned pointer to a local string. Test empty text, accents, a supplementary character, combining marks, embedded nulls if allowed, non-default locale, and an encoding round-trip. Include malformed input where the application defines a rejection policy.

**Migration decision:** an old `PChar` call that compiled in Delphi 5 is not evidence it still calls an ANSI API correctly. Match the declared foreign function's exact character width and buffer length units.

## Sources and interpretation

H: *Object Pascal Handbook 11*, ch. 6, printed pp. 165–199 (PDF 166–200), especially pp. 167–170, 178–185, 192–199; ch. 14 pp. 412–416 and ch. 18 pp. 513–515. Boundary-validation and test recommendations are editorial applications of these representation rules.
