# 3A — True 3D Arc Through Three Points

Copyright (c) 2026 Dzmitry Tsiapkin. All rights reserved.

**Author:** Dzmitry Tsiapkin

`3A` creates one native AutoCAD ARC through three spatial points: start, intermediate point, end. It computes the plane from those points without changing your UCS. Useful for surveyed curved edges, tilted steelwork, infrastructure and as-built drawings. No tessellation, external libraries, ActiveX or dependency on 3C/3DSNAP.

## Install and use

1. Keep `3A.lsp` in a local folder approved by your CAD security settings.
2. Run `APPLOAD` and select `3A.lsp`.
3. Run `3A` and answer:

```text
Specify arc start point:
Specify point on arc:
Specify arc end point:
```

Pick accurate 3D object snaps or enter full X,Y,Z coordinates. Typed coordinates are in the current UCS. The command converts them to WCS; it does not require UCS alignment to the arc plane. The entity uses the current layer. Enter cancels an incomplete selection; ESC is handled by the command's local error handler.

Point order defines the path **P1 → P2 → P3**. P2 is a point on the arc, not its center. A point on the longer portion of the circle produces a major arc. Reversing the order reverses the normal and traversal while retaining the same geometric path. Nearly full arcs are supported when the remaining gap is numerically resolvable; identical start/end points are rejected.

The standard ARC command offers several constructions, including three points, center/radius and tangent-related options ([Autodesk: About Arcs](https://help.autodesk.com/cloudhelp/2025/ENU/AutoCAD-Core/files/GUID-C78378FD-4704-4D0A-9D86-82FCBB118097.htm)). 3A provides a dedicated three-spatial-point workflow with explicit WCS plane calculation, direction selection and post-creation checks. It does not implement the other ARC options or change the built-in command.

## Verification and limitations

Executed on 2026-10-27 in **AutoCAD 2022 Core Console**, native AutoLISP engine: 546 regression assertions, 36 command integration assertions, and a separate successful DWG reopen/count check. No failures. See [Testing](TESTING.md) and [Geometry and validation](VALIDATION.md) for scope.

Interactive APPLOAD, mouse snaps, physical ESC, graphical editing and BricsCAD remain manual verification items. Other AutoCAD versions, LT and macOS were not tested. Basic AutoLISP/entmake/trans are used to support portability, not to claim verified compatibility.

The command preserves UCS and system variables; normal point input may update CAD's LASTPOINT. Existing snap/projection settings still govern what `getpoint` receives: when snapping true 3D geometry, check OSNAPZ and use appropriate object snaps. 3A cannot recover a Z coordinate already replaced by input settings.

Duplicate, poorly resolved and nearly collinear points are rejected. Supported numeric bounds are documented in VALIDATION.md. Tiny or huge geometry may be unsuitable for display or downstream CAD operations even when its arithmetic checks pass. Three noisy surveyed points determine an exact circular arc; this is not a multi-point best-fit tool.

Load `3A-tests.lsp` separately and run `3ATEST` in a scratch drawing for regression checks. Production use requires only `3A.lsp`.

## Repository files

| File | Purpose |
|---|---|
| [3A.lsp](3A.lsp) | Standalone production command |
| [3A-tests.lsp](3A-tests.lsp) | Native AutoLISP regression suite |
| [TESTING.md](TESTING.md) | Executed results and manual acceptance procedures |
| [VALIDATION.md](VALIDATION.md) | Mathematics, direction handling and numerical limits |
| .gitignore | Excludes local CAD drawings, backups and development artifacts |

## Copyright and availability

Copyright (c) 2026 Dzmitry Tsiapkin. All rights reserved.

The source code is publicly available for inspection. No explicit license for copying, modification, redistribution, or commercial reuse is granted with this release.
