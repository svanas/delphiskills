---
name: delphi-components
description: Create or maintain Delphi components, streamed properties, notifications, custom controls, frames, and runtime/design-time package boundaries.
---

# Reusable components and design-time packages

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Choose the smallest correct base

Use a plain class for nonvisual logic that needs no component ownership or streaming. Use `TPersistent` for assignable/streamable subobjects, `TComponent` for owner-managed nonvisual components, and the appropriate VCL or FMX control base for visual behavior. A frame can reuse a composition without inventing a new component class. Do not mix VCL and FMX control inheritance. [H ch. 18; M ch. 13; E ch. 7]

Preserve the inherited constructor contract; component constructors generally override `Create(AOwner: TComponent)`. The IDE and streaming machinery can construct via class references, so a hidden constructor is not equivalent to an override.

## Properties are a persistence contract

Choose published properties deliberately: names and types are part of DFM/FMX compatibility. Set runtime defaults in the constructor. The `default` specifier controls whether a value is streamed; it does not initialize the field. Use setters that validate changes, avoid redundant work, and invalidate/recompute only when required. [M PDF 635]

For a persistent subobject, normally create one owned instance and implement assignment of its values rather than silently replacing the pointer supplied by a caller. Implement `Assign` with type checks and delegate unsupported source types to the inherited method. Do not assume `Assign` always means a deep clone of every referenced object.

Property setters can run during loading before all sibling components are connected. Use `Loaded` for work requiring complete streaming and inspect `csLoading`, `csDesigning`, or destruction state where the behavior needs it. Avoid opening network connections, starting threads, or writing files simply because a component appears on a design surface.

## References and events

For a property referencing another component, register for its destruction notification and clear the reference in `Notification` when removed. Remove obsolete notification relationships when replacing references if the contract requires it. Call inherited notification logic. This is distinct from owning the referenced component. [M PDF 288; H ch. 18]

Declare events with signatures suitable for the Object Inspector, check assignment before calling, and document whether handlers may change or destroy the sender. Guard reentrant state transitions. A published event is normally a method pointer; do not replace it with `TProc` without evaluating designer compatibility and callback lifetime.

## Runtime versus design-time

Put the reusable component and runtime dependencies in a runtime package. Keep registration routines, property editors, component editors, and IDE APIs in a design-time package that depends on the runtime package. Do not pull IDE-only units into deployed applications. Inspect `.dpk` requires/contains and `.dproj` configurations together.

Package binaries depend on compiler, target, and package dependencies. Build/install the package for the actual IDE architecture; application bitness and IDE bitness are separate facts. Do not derive an installation directory or package suffix from the marketing name. The 2025 annotated book explicitly calls out bitness matching for loaded binaries. [M ch. 14 PDF 675; current version documentation]

When upgrading an existing component, preserve streamed names or supply a deliberate migration. Test a form saved with the previous version, not just a newly dropped instance with default properties.

## Verification

Test create/free with nil and non-nil owners, partial constructor failure, property changes before/after loading, referenced-component removal, streaming round trips, inherited forms, and repeated package loading where feasible. Check that a runtime application builds without design-time packages. Report designer validation as unperformed when no matching IDE is available.

## Sources and interpretation

H: *Object Pascal Handbook 11*, ch. 10 pp. 286–300; ch. 18 pp. 503–507. M: *Mastering Delphi 5, 2025 Annotated Edition*, ch. 13 (PDF 612–667), especially PDF 635; ch. 14 (PDF 668–710), especially PDF 675 footnote 352; PDF 288 for notification. E: *Expert Delphi* ch. 7 pp. 189–198. Package separation and compatibility checks are editorial applications of the component model.
