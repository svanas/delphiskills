---
name: delphi-rtti
description: Use Delphi RTTI, TValue, custom attributes, method invocation, and reflection-based mapping with correct lifetime and type checks.
---

# Reflection, attributes, and serialization contracts

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## First decide whether reflection is needed

Use ordinary typed code when the set of types and operations is known and small. RTTI fits generic serializers, inspectors, mapping, and framework extension points where runtime metadata actually removes repetition. Do not replace direct property access with reflection merely to shorten source. [H ch. 16; F “Enhanced RTTI”]

Identify classic published RTTI versus extended RTTI. Inspect relevant `{$RTTI}` and linking settings and the actual target types; do not assume all private fields or all methods are emitted. Attributes add metadata; they do not execute application behavior by themselves.

## Context and metadata lifetime

Maintain a deliberate `TRttiContext` lifetime for metadata access. RTTI objects obtained through it are managed by the RTTI infrastructure; do not individually free `TRttiType`, `TRttiProperty`, or attribute instances returned by enumeration. A longer-lived context can support a metadata cache, but cached pointers need a valid context and loaded-module lifetime. Do not carry metadata past package unloading. [H pp. 454–460]

For a local inspection, create the context, perform the complete inspection in its lifetime, and release the context appropriately. For a shared cache, design its initialization, synchronization, invalidation, and shutdown rather than creating ad hoc global RTTI pointers.

## Validate lookup and values

Handle a missing type/member explicitly. Check property readability/writability, method parameters, result kind, and instance requirements before accessing or invoking. An input string naming a method is not authorization to invoke any reflected method. Map external names to an intentional allowlist of operations or fields.

Use `TValue` with a known expected type; distinguish empty values, nil object references, null-like protocol values, and failed conversions. `TValue` wrapping an ordinary object does not automatically own that object. An array of `TValue` arguments still needs the correct argument order and passing convention. Reflection is not a safe way to ignore a static type mismatch. [H pp. 461–465]

## Attributes and mapping

Keep attribute constructors lightweight and deterministic. Metadata consumers may instantiate attributes while inspecting types, so avoid starting I/O or changing application state in those constructors. Decide inheritance and duplicate-attribute policies; do not silently assume a property-level attribute replaces all inherited behavior.

For serialization, define which members participate, their external names, required/null rules, versioning, and object ownership on deserialization. Preserve public wire names when renaming Delphi symbols. Do not deserialize untrusted arbitrary class names into unrestricted objects just because RTTI can locate them.

## Performance and verification

Cache resolved metadata only when measurement justifies it and its lifetime is controlled. Repeated `GetTypes` scanning in a hot loop is usually a poor substitute for a targeted lookup. Measure reflection cost separately from parsing, allocation, and I/O. [H ch. 16; F pp. 84–85, 105–108]

Test missing metadata, read-only properties, incompatible `TValue` types, inherited members, attribute presence/absence, nil instances, and unloaded/reloaded packages if supported. For a mapper, test older serialized data and fields whose language identifier differs from the external name. For invocation, test exceptions thrown by the target and ensure the caller reports the actual failure rather than swallowing it.

## Sources and interpretation

H: *Object Pascal Handbook 11*, ch. 16 pp. 449–486 (PDF 450–487), especially pp. 451–458, 461–469, 473–485. F: *Code Faster in Delphi*, pp. 105–108 (PDF 108–111). E: *Expert Delphi*, ch. 2 pp. 61–65. Cache design, allowlisting, and compatibility tests are editorial applications of the RTTI model.
