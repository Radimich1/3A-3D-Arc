# Geometry and validation

Copyright (c) 2026 Dzmitry Tsiapkin. All rights reserved.

## Reused mathematics

The existing working 3C implementation was inspected. Its vector operations, scaled Euclidean norm, longest-edge normalization, circumcenter formula, dimensionless area rejection and coordinate-aware residual tolerances were adapted into independent `a3:` functions. 3A loads no 3C files and changes none of them.

For WCS points p,q,r, set u=q-p, v=r-p, s=max of the three edge lengths; a=u/s, b=v/s, N=a×b. The center offset in normalized coordinates is:

```text
h = (|a|² (b×N) + |b|² (N×a)) / (2 |N|²)
center = p + s h
radius = |center-p|
normal = N / |N|
```

The norm scales by the largest component before squaring. Input coordinates must be numeric three-element lists with absolute components <=1e100; each edge and the radius must exceed 1e-100. Computed center components and radius must also be <=1e100. These conservative bounds avoid overflow/underflow in intermediate arithmetic; they are not accuracy guarantees throughout that entire range.

With epsilon=2.220446049250313e-16 and M the largest absolute input coordinate, every edge must exceed 64 epsilon M. Reject |N|<=1e-8 or |N|s<=64 epsilon M. This deliberately rejects ill-conditioned arcs, including very shallow arcs and almost-closed arcs whose gap is too small.

The linear tolerance is:

```text
tol = 128 epsilon maxAbs(p,q,r,center) + 1e-9 max(s,radius)
angularBudget = max(1e-12, 4 tol/radius) radians
```

This is a numerical residual budget in drawing units, not a survey accuracy specification. All point-to-plane and radial residuals must be <=tol; unit-normal error must be <=1e-10. Edge perpendicularity is also checked.

## Coordinates and direction

`getpoint` values are converted UCS→WCS with `trans`. The center is stored in entity OCS using `trans center 0 normal`. Radial displacements use `trans displacement 0 normal T`; atan(y,x) then yields angles normalized to [0,2pi). Using displacements avoids translation cancellation when calculating angles.

Native ARC progresses counterclockwise around its extrusion normal. With normalized modulo arithmetic, the middle offset must be strictly inside the start/end sweep, separated from both endpoints by angularBudget. The sweep must also be separated from zero and 2pi by that budget. If the initial normal does not satisfy this, its sign is reversed and angles recalculated; unresolved cases are rejected. Major arcs are retained, never automatically shortened.

DXF group 10 is the OCS center, 40 the radius, 50/51 the angles and 210 the normal. Angles supplied through AutoLISP are **radians**, although textual DXF files use degrees. References: [ARC entity data](https://help.autodesk.com/cloudhelp/2020/ENU/AutoCAD-DXF/files/GUID-0B14D8F1-0EBA-44BF-9108-57D8CE614BC8.htm), [DXF group codes](https://help.autodesk.com/cloudhelp/2024/ENU/AutoCAD-DXF/files/GUID-3F0380A5-1C15-464D-BC66-2C5F094BCFB9.htm).

## Post-creation verification and rollback

Read back the actual entity, require ARC type, supported positive radius, valid center and unit normal matching the solution. Check center/radius agreement, all three radial and plane residuals, reconstruct actual start/end positions from stored angles and compare to P1/P3. Evaluate P2 against the actual stored angular sweep. Merely being on the underlying circle is insufficient.

Only the newly appended entity is eligible for rollback. The previous entlast is recorded and excluded from deletion; the local error handler also checks this guard. No existing entities or settings are modified by production code.

## Executed evidence, 2026-10-27

AutoCAD 2022 Core Console executed both source files successfully. 273 assertions passed in World UCS and 273 in translated/rotated UCS. These include 12 fixed positive fixtures, eight rejection cases, a UCS round-trip check and 120 deterministic generated arcs per run. Each positive fixture checks actual endpoints, center/radius, expected sweep length, production post-validation, and rejection after swapping native start/end angles.

Independent fixture expectations check endpoints and expected sweep; production validation additionally checks P2 membership. Tests use a looser independent comparison budget of max(radius*1e-7,5e-14*coordinate magnitude), while the production verifier still applies its own tighter tolerance. Generated cases use an inclined orthonormal frame and alternating sweep signs.

Command integration: eight successful commands in two UCS frames, Enter at each of three prompts, collinear rejection, simulated creation failure, and simulated post-validation failure: 36 assertions passed. Settings and existing entity counts were checked. A saved DWG was reopened and all eight native ARCs were present; this persistence check counted entity types, not every geometric field after reopening.

Physical ESC, mouse snapping, GUI rendering/editing and BricsCAD were not exercised. These remain the manual checks in TESTING.md.
