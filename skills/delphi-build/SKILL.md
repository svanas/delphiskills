---
name: delphi-build
description: Discover Delphi toolchains and build projects or packages with their real configurations, diagnose errors, and report verified compilation accurately.
---

# Compiler discovery, builds, and deployment checks

For Delphi 11 Alexandria and newer. If not already loaded, read [the shared workflow](../delphi/SKILL.md) once; do not recursively reload it. Apply only the parts relevant to the requested change.

## Discover before invoking

Read `.dproj`, `.groupproj`, `.dpr`, `.dpk`, include files, build scripts, CI configuration, and dependency instructions. Record compiler/version, configuration, platform, defines, search paths, runtime-package settings, and external dependencies. Read existing build events before running them: project builds can invoke arbitrary pre/post-build commands.

Find an installed licensed Embarcadero toolchain and the project's established invocation. A shell named “developer prompt,” `dotnet`, MSBuild alone, or Free Pascal is not proof that Delphi compilation is available. Free Pascal can help specific compatible projects but is not validation of Embarcadero Delphi 11 behavior.

Use the installed RAD Studio command environment or its `rsvars.bat` to initialize variables in the **same command session** as MSBuild. Do not hard-code Studio directory numbers from `CompilerVersion`: Alexandria compiler 35.0, Athens 36.0, and Florence 37.0 are compiler identifiers, not a universal folder-number formula. Florence changed internal version numbering. [Official compiler/build documentation]

## Respect project configuration

Prefer MSBuild on the `.dproj`/`.groupproj` when that is the project workflow; it carries configuration inheritance, references, resources, and compiler options that a bare `dcc32` invocation might miss. Use direct compiler calls only when their options/search paths are deliberately reproduced.

Example for an already configured RAD Studio Windows command prompt, after replacing project/configuration/platform with values actually present:

```bat
msbuild "Example.dproj" /t:Build /p:Config=Debug /p:Platform=Win32
```

This is an invocation template, not a claim that `Example.dproj` exists. Do not pass Unix quoting or shell syntax to `cmd.exe`. Capture the exit code and useful diagnostics. Select `Make`, `Build`, or `Clean` according to the project's targets and intended scope; do not assume “build” proves every stale artifact was regenerated. Clean only known generated outputs when needed.

Separate DCU/DCP/BPL/EXE outputs by compiler, platform, and configuration. Do not “fix” a missing dependency by copying an incompatible DCU from another Delphi release. Preserve resource files and deployment metadata.

## Diagnose from the first meaningful error

Classify the failure before editing code:

| Failure | Inspect first |
| --- | --- |
| Unit not found | Active search paths, namespace scopes, package/source dependency, selected platform |
| Built with a different version | Stale DCU/DCP or mixed compiler/dependency outputs |
| Undeclared identifier | Target version, unit scope, conditional defines, actual API availability |
| Type/signature mismatch | Public declaration, overload choice, generic constraints, platform width |
| Resource/streaming error | DFM/FMX class/property/event bindings and linked resource |
| Link/load failure | Library bitness, exports, runtime packages, deployment location |
| SDK/toolchain missing | Selected target installation, SDK registration, remote host configuration |

Do not make speculative source edits to compensate for a missing SDK or license. Do not suppress all warnings to obtain a green build. Review newly introduced warnings and distinguish them from documented existing ones.

## Build matrix and version gates

For code promised to support Delphi 11+, validate the minimum supported compiler where available, then newer supported versions affected by the change. A Florence-only build cannot prove Alexandria compatibility. Build affected target/configuration branches; if resources or package architecture changed, include runtime loading/designer checks where possible.

Keep 12+ wizard features and 13+ syntax out of an unconditional 11-compatible path. Deployment is separate from compilation: mobile SDKs, signing, entitlements, library files, styles, and database assets must match the intended target. Check the current platform-status documentation during deployment rather than preserving stale version numbers from books.

## Completion report

State exactly what ran: compiler version, project, configuration, platform, exit status, and relevant tests. If the compiler is absent, perform source/configuration checks and say **not compiled**. Give the intended command and missing prerequisite without inventing an executable path. If build passes but tests, designer loading, or deployment were unavailable, list those independently. Never report all Delphi versions/platforms verified from one build.

## Sources and interpretation

E: *Expert Delphi*, ch. 1 pp. 3–32 and ch. 14 pp. 367–382. H: *Object Pascal Handbook 11*, ch. 1 pp. 37–47. M: *Mastering Delphi 5, 2025 Annotated Edition*, ch. 14 PDF 668–710 and ch. 18 PDF 833–867. Supplements: [MSBuild](https://docwiki.embarcadero.com/RADStudio/Florence/en/MSBuild_Overview), [Building a Project Using an MSBuild Command](https://docwiki.embarcadero.com/RADStudio/Florence/en/Building_a_Project_Using_an_MSBuild_Command), [Compiler Versions](https://docwiki.embarcadero.com/RADStudio/Florence/en/Compiler_Versions), [Florence changes](https://docwiki.embarcadero.com/RADStudio/Florence/en/What%27s_New), [Platform Status](https://docwiki.embarcadero.com/PlatformStatus/en/Main_Page). Discovery and reporting procedures are editorial workflow guidance.
