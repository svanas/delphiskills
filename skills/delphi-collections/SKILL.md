---
name: delphi-collections
description: Select and safely use Delphi generic lists, object containers, dictionaries, arrays, constraints, and comparers.
---

# Generics, collections, and value semantics

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Select by semantics

Use an array for indexed sequences, `TList<T>` for a resizable sequence, `TDictionary<TKey,TValue>` for key lookup, and an owning object collection when destruction is part of the container's job. Select for behavior before micro-optimizing. A dictionary is not an ordered list, and a string list is not merely an obsolete dictionary: `TStringList` also supplies ordering, duplicates, name/value behavior, text loading, and notifications. [H ch. 14, 18; F “Generic Collections”]

Generics are type-safe but not automatically covariant. A list of a descendant is not generally a list of its ancestor. Do not bypass that restriction by casting the container. Choose a common abstraction, an iterator, or a deliberate copy when the API needs a different element type. Use constraints only for operations the implementation actually requires; `class` does not itself guarantee a public parameterless constructor. [H ch. 14]

## Array assignment is not isolation

Assigning a dynamic array aliases its storage. Unlike normal string writes, changing an element does not invoke copy-on-write to isolate the arrays. Use `Copy` for a separate outer array; nested arrays and object references can still be shared. A by-value dynamic-array parameter is therefore not a deep immutable snapshot. Protect shared arrays across threads or construct an independent snapshot with the required depth. [H ch. 5 pp. 129–134]

Use `Low`, `High`, or `Length` according to the actual array type. Static arrays can start at nonzero ordinal bounds; dynamic arrays start at zero. Check empty sequences before computing averages or taking element addresses. Do not keep element pointers across reallocation.

## Make ownership visible

Construct an owning object list with an explicit ownership choice, such as `TObjectList<TEntry>.Create(True)`, so readers can see the policy. `Delete`, `Remove`, `Clear`, and destruction can free owned items. Use the documented extraction operation to transfer an item without freeing it. Do not insert one object into multiple independently owning lists.

For `TObjectDictionary`, examine `doOwnsKeys` and `doOwnsValues` independently. A plain generic dictionary does not automatically free object-valued entries. Define who owns a newly constructed item if insertion fails because a key already exists. Review overwrite behavior before using `AddOrSetValue`: replacing an owned value may destroy the old object. Do not retain its borrowed pointer. [H “Object Containers” and “Using a Generic Dictionary”]

## Keys and comparers

Use `TryGetValue` when absence is normal rather than an exception-driven lookup. `Add` should expose unexpected duplicates; `AddOrSetValue` expresses intentional replacement. Do not mutate the fields contributing to a key's hash/equality while it is stored. Equality and hashing must agree: equal keys must hash identically.

For sorted data, use a consistent comparer with an explicit case and null policy. Avoid `Result := Left - Right` for integer comparison when subtraction can overflow; compare relationally. A comparer supplied as an anonymous method must not depend on mutable captured state that changes during sorting. Do not assume an unstable sort preserves equal-key order. [H ch. 14; F printed pp. 86–90]

## Mutation and cost

Do not structurally mutate a collection while enumerating it unless its documented enumerator permits that. Collect keys to remove, use an appropriate index traversal, or rebuild the collection. A `for-in` record variable is a copy; modifying it may not write back to the collection. Keep lookup and update atomic under a shared lock when concurrent callers depend on the combination.

Reserve capacity only when a meaningful size estimate exists. Measure real data distributions before replacing one structure with another; tiny collections, ordering requirements, allocations, and expensive comparers can dominate asymptotic lookup benefits.

## Verification

Test empty and singleton collections, duplicate keys, missing keys, nil values when allowed, hash collisions, ownership on removal/replacement, enumeration under attempted mutation, and boundary values in comparers. For an array snapshot, mutate the source afterward and assert exactly which levels remain shared.

## Sources and interpretation

H: *Object Pascal Handbook 11*, ch. 5 pp. 125–137; ch. 14 pp. 383–429, especially 398–416; ch. 18 pp. 513–515. F: Alister Christie, *Code Faster in Delphi* (2020), printed pp. 74–76 and 86–90 (PDF pages +3). Ownership flags should be verified against the target RTL; the handbook's abbreviated object-container discussion is not an exhaustive constructor contract. Comparison, failure-path, and test guidance is editorial synthesis.
