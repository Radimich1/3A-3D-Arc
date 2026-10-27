# Testing 3A

Copyright (c) 2026 Dzmitry Tsiapkin. All rights reserved.

## Automated regression

In a scratch drawing, APPLOAD `3A.lsp`, then `3A-tests.lsp`; run `3ATEST`. Expected result: **273 passed; 0 failed**. It creates and removes its own test entities, leaves UCS/settings unchanged, and does not require other projects. Undo history may contain test operations. Test results are printed to the command line.

For non-default UCS coverage, set a translated and rotated UCS in the scratch drawing, then run `3ATEST` again. In addition, run command `3A` with typed coordinates in that UCS and compare WCS endpoints. The regression runner uses WCS fixtures without resetting your active UCS.

Fixtures cover XY, XZ, YZ, an inclined plane, minor/semicircle/major/nearly-full arcs, clockwise/reversed input, survey coordinates (6580000,670000,35), radii 1e-9, 1e-80 and 1e80, duplicate/collinear/nearly-collinear/unresolved points, and malformed helper inputs. Generated arcs cover both directions and angle wraparound. Positive tests read actual native ARC geometry and deliberately swap stored endpoints to ensure the wrong sweep is rejected.

## Executed native tests

On 2026-10-27, AutoCAD 2022 Core Console completed:

| Check | Result |
|---|---|
| Regression in World UCS | 273 passed, 0 failed |
| Regression in translated/rotated UCS | 273 passed, 0 failed |
| Scripted production command integration | 36 passed, 0 failed |
| Saved DWG reopened: eight native ARCs present | Passed |

These are actual CAD-engine tests, not an emulated Lisp evaluation. Integration used isolated profiles and scratch drawings; the extra integration harness is development evidence, not required to run 3ATEST. No BricsCAD or GUI execution is claimed.

## Manual acceptance checklist (remaining)

Use a scratch drawing, save its initial state and record UCS, layer and snap settings. For typed numeric examples use World UCS, disable snap overrides as appropriate, and enter full coordinates. Return settings after your manual tests.

1. APPLOAD production source alone. Run `3A`: (10,0,0), (0,10,0), (-10,0,0). Check ARC type, radius 10 and semicircle through positive Y using LIST/Properties.
2. Major arc: (10,0,0), (-10,0,0), (0,-10,0). Expect 270 degrees about +Z. Reverse points: (0,-10,0), (-10,0,0), (10,0,0). Expect the same path with reversed traversal/normal.
3. Inclined: (5,7,6), (-3,5,8), (-3,-1,2). Expect center (1,3,4), radius 6; orbit the view and snap to the three locations. Repeat from a rotated UCS using the same physical WCS points. Do not retype WCS numbers as if they were rotated-UCS numbers.
4. Check XZ: (10,0,0), (0,0,10), (-10,0,0); YZ: (0,10,0), (0,0,10), (0,-10,0). Inspect plane normals and endpoints.
5. Nearly full: (10,0,0), (-10,0,0), (9.99999995,-0.001,0). Expect nearly 360 degrees with a small gap, not a short arc.
6. Pick real 3D object snaps on inclined geometry; confirm Z is preserved. Inspect OSNAPZ if points are projected to an elevation. Check endpoint and midpoint-on-path visually and with CAD inquiry tools.
7. Press ESC separately at each prompt; no new entity should remain, previous geometry and UCS/settings should be unchanged. Enter cancellation was already scripted at all three prompts; repeat interactively if desired.
8. Reject duplicate points, (0,0,0)/(1,0,0)/(2,0,0), and (0,0,0)/(1,0,0)/(2,1e-10,0). Existing objects must remain intact.
9. COPY, MOVE, ROTATE, undo, save/reopen, then inspect ARC type and geometry. Repeat on locked layers and in paper space if these are part of your workflow.
10. Run the same checks in each target BricsCAD/AutoCAD release before asserting compatibility. Record application version and actual results.
