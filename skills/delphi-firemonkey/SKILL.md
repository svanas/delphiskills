---
name: delphi-firemonkey
description: Build and review Delphi FireMonkey layouts, styles, mobile services, lifecycle handling, 2D/3D scenes, and platform-specific behavior.
---

# FireMonkey, mobile lifecycle, and graphics

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Separate shared logic from platform behavior

Identify the real targets and compiler updates first. A successful Windows FMX build does not validate Android permissions, iOS lifecycle, macOS deployment, or Linux GUI support. Keep domain logic in RTL-only units where possible and isolate platform services/bridges behind focused adapters. Do not import `Vcl.*` or Windows API units into shared FMX code. [E ch. 5, 8–9]

Use `TPlatformServices.Current.SupportsPlatformService` for optional services and handle absence. A compile-time platform symbol selects available code; a runtime service check selects capabilities on the running device. Neither establishes permission to access a sensor or camera. Handle unsupported, denied, canceled, and temporarily unavailable results separately.

## Layout and reusable UI

Use alignment, anchors, margins, padding, and layout containers to express relationships. Choose a flow/grid/scaled layout for its actual behavior, and test text growth, orientation changes, virtual keyboard, and display scale. A scaled layout scales content; it is not automatically the right answer for every device size. Frames encapsulate reusable compositions; inherited forms/views require attention to inherited resource overrides. [E ch. 5, 7, 9]

Styles can change the internal visual tree. Do not assume a style resource child always exists or has the same class on every platform. Resolve styled objects at the correct style lifecycle point, tolerate missing optional elements, and release cached references when styles change. Include required style resources in deployment. Keep business logic independent of style-object names. [E ch. 7]

## Lifecycle and delayed work

Handle application activation/background transitions using supported FMX services. Save important state at meaningful lifecycle transitions and during normal work; do not depend exclusively on a final termination callback. Pause or adjust timers, sensors, animations, and pending operations when appropriate, then restore them coherently on resume. Re-registering handlers repeatedly must not produce duplicate subscriptions. Unregister borrowed callbacks before their owner dies. [E ch. 8 pp. 208–210]

UI work belongs on the main thread. Long work must not block a UI timer or event handler. Use the concurrency skill for queue timing, exception observation, cancellation, and form closure. A sensor or HTTP callback may arrive after the user leaves a screen.

## Files, devices, and bridges

Use `TPath` and platform-appropriate writable storage rather than hard-coded Windows paths. Separate deployment of read-only assets from the writable location of databases/settings. Verify the deployment mapping and first-run copying behavior on the actual target. Never assume the desktop working directory is the mobile application's document directory. [E ch. 3, 8, 10, 14]

Before adding Java/JNI or Objective-C bridge declarations, inspect the target SDK and existing Delphi bridge units. Match types and ownership conventions; keep conditional imports within platform adapters. Permissions, entitlements, SDK levels, app-store requirements, and device APIs evolve independently of Delphi syntax. Look them up for the exact Delphi update and target OS when implementing deployment; do not reuse historical book values as current requirements.

## Graphics and timing

Store scene state and render from the proper paint/render path. Pair canvas begin/end operations when required by the API. Use elapsed time for animation steps instead of assuming every timer interval fires exactly on schedule. For 3D, keep transforms, camera, lighting, material, and model coordinate conventions explicit. Check picking/input in the appropriate coordinate space, and test mixed 2D/3D overlays on devices. Avoid rebuilding textures or geometry every frame without a measured reason. [E ch. 5–6]

## Verification

Exercise unsupported services, denied permissions, background/resume, low-memory behavior where testable, screen transitions while work is pending, rotation/resize, style changes, and a clean-device deployment. Mark each actual tested platform; never turn “cross-platform source” into a claim that every target was run.

## Sources and interpretation

E: *Expert Delphi*, second edition, ch. 5 pp. 107–144; ch. 6 pp. 145–174; ch. 7 pp. 175–201; ch. 8 pp. 203–233 (lifecycle PDF 231–233); ch. 9 pp. 235–262; ch. 14 pp. 367–382. Platform-validation and callback-shutdown guidance is editorial synthesis. Consult [Embarcadero Platform Status](https://docwiki.embarcadero.com/PlatformStatus/en/Main_Page) for target toolchain requirements instead of freezing book-era SDK numbers.
