---
name: delphi-vcl
description: Review or change Delphi VCL forms, DFM resources, actions, component ownership, event handlers, painting, and Windows UI lifecycle.
---

# VCL forms, resources, and application behavior

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Treat a form as code plus a resource

Inspect `.pas`, `.dfm`, ancestor forms/frames, and the project's creation order together. A compile can succeed even though streaming fails when a form is constructed. A component rename can affect the field declaration, DFM object name, event bindings, references in other forms, RTTI lookup, and external persistence. Preserve `{$R *.dfm}` and the resource/class relationship. [M ch. 4–8; F “Editing the Form's Source”]

When changing a component class, check every persisted property and event signature against the new class. Preserve `object`, `inherited`, and `inline` distinctions and inherited overrides. Do not delete an unknown property solely to silence the loader without understanding its behavior. Binary DFM resources need the established conversion workflow; do not run text replacement on them.

## Ownership and form lifetime

`Owner` is the component ownership relationship; `Parent` is the visual/container relationship. Setting one is not a general substitute for understanding the other. A component owned by a form is usually released during the form's destruction. A component created with nil owner needs another explicit owner/release path. Borrowed cross-component references need notification or another invalidation contract. [H ch. 18; M ch. 4, 6]

Decide whether a form is auto-created, created on demand, modal, reusable modeless, or self-releasing. For a locally owned modal form, use a `try/finally` around `ShowModal` and free it once. For a modeless form using `caFree`, update references in its destruction path and avoid a second manual free. `Close` does not universally mean destruction. Do not free a form in an event while other code still assumes it exists; understand deferred `Release` behavior where appropriate.

Avoid implementation code that refers to its own global `Form1` variable. Use `Self` and instance fields so a second instance works. Pass dependencies or expose intentional properties rather than reaching through arbitrary global forms. Move business operations into callable methods/services; event handlers should adapt UI input and display results. [H “15 Tips About Mixing RAD and OOP”]

## UI state and reentrancy

Use actions for shared command execution and enablement when the application already uses them. Keep action-update handlers inexpensive and free of surprising business side effects. Be careful with event recursion when setters change controls that fire the same events. For temporary event suppression, restore the prior state in `finally` rather than always enabling it afterward.

Use layout constraints, alignment, anchors, and the project's DPI approach instead of fixed pixel arithmetic copied from old screenshots. Test different DPI settings, fonts, long translated captions, keyboard navigation, and tab order. If a control's handle can be recreated, do not treat a cached `Handle` as permanent. Isolate Windows-specific message handling from shared domain code. [M ch. 5–8; editorial modern UI checks]

## Painting and graphics

Draw persistent content from the control's paint path, backed by state. Drawing once onto a canvas outside repaint handling is not persistent across invalidation, exposure, or resizing. In a property setter, update state and invalidate when the value actually changes. Release bitmaps, pens, brushes, and other resources according to their API contracts. Avoid per-paint allocations when profiling shows they dominate; do not cache resources past changes that invalidate them. [M ch. 13, 22]

Do not access ordinary VCL controls from background workers. Use the concurrency skill for completion delivery and shutdown. Historical canvas-lock examples are not a blanket license to make UI calls from threads.

## Verification

Open/load affected forms, including inherited forms, then exercise their actual creation and destruction paths. Check DFM names and event bindings, design-time default behavior, modal result handling, repeat-open behavior, resizing, DPI, and shutdown during pending work. Report separately if the compiler was available but the IDE/designer or GUI runtime was not.

## Sources and interpretation

H: *Object Pascal Handbook 11*, ch. 10 pp. 303–309 and ch. 18 pp. 502–507. M: *Mastering Delphi 5, 2025 Annotated Edition*, ch. 4–8 (PDF 162–400), ch. 13 pp. 612–667 and ch. 22 pp. 1060 onward; concrete lifecycle anchor: PDF 288–289, including FreeNotification. F: *Code Faster in Delphi*, pp. 42–57 (PDF 45–60). DPI, reentrancy, and verification procedures are editorial modernization, not claims that Delphi 5 supported current UI features.
