# Scordelis-Lo Roof — LS-DYNA Keyword Decks

The classic **Scordelis-Lo cylindrical shell roof** benchmark (Scordelis & Lo,
1964), as runnable LS-DYNA keyword decks. The decks are taken from the
`qashell/09-scordelis-lo` verification case of the
[sale](https://github.com/hao94588/sale) explicit shell code, converted for
LS-DYNA with a single change: `*BOUNDARY_SPC` → `*BOUNDARY_SPC_NODE`
(the two keywords carry identical data; LS-DYNA only accepts the `_NODE`
form). Everything else — nodes, elements, material, loads, curves — is
unchanged from the sale original.

## Model

- **Geometry:** cylindrical shell segment, radius **R = 25**, axial length
  **50** (x = 0…50), included angle **80°** (θ = −40°…+40°), thickness
  **0.25**. The crown is at (y = 0, z = 25); the two straight free edges are
  at (y = ±16.06969, z = 19.151111).
- **Material:** `*MAT_ELASTIC`, **E = 432**, **ν = 0**, density **0.1**.
  The standard values (E = 4.32e8, q = −90) are scaled by 1e6, so the
  dimensionless response q/E = 2.083e-7 is unchanged.
- **Boundary conditions — the important part:**
  - **Diaphragm ends** (x = 0 and x = 50, the two arc edges): the end
    cross-section is rigid in its own plane but free out of plane.
    Implemented as `*BOUNDARY_SPC_NODE` with **uy = uz = 0, ux free, all
    rotations free** (34 nodes, 17 per end). The diaphragm keeps the
    cross-section shape while letting the shell slide axially and rotate
    freely at the ends.
  - **Straight edges** (θ = ±40°, the two longitudinal edges): completely
    free.
- **Load:** uniform vertical pressure **q = −9e-5** (global −z, i.e.
  self-weight direction), applied as consistent lumped nodal forces
  (`*LOAD_NODE_POINT`, DOF 3): interior nodes −0.0006136, edge nodes half,
  corner nodes a quarter. Total load −0.15708 = q × area
  (50 × 25 × 80° in radians = 1745.33).
- **Loading history:** a smoothstep curve ramps the load from 0 to 1 over
  **0–500 s** (0–600 s for the 64×64 decks, with a 500–600 s hold). This is
  a quasi-static ramp; the model is undamped, so judge the result on the
  time-averaged window, not on a single instant during the ramp.

## Reference value

Vertical displacement at the **midpoint of the free edge** (x = 25,
θ = −40° edge), magnitude **0.3024** (downward). Monitor nodes by mesh:

| mesh | monitor node |
|------|--------------|
| 8×8   | 5  |
| 16×16 | 9  |
| 32×32 | 17 |
| 64×64 | 33 |

## Files

| file | what it is |
|------|------------|
| `scordelis-lo-{8,16,32,64}x{8,16,32,64}.k` | official decks, full load. **Nonlinear — do not compare directly to 0.3024.** |
| `scordelis-lo-{8,16,32,64}x{8,16,32,64}-low-slow.k` | linear-acceptance decks: total load ×0.01. Divide the measured displacement by 0.01 to get the per-unit value for comparison with 0.3024. |
| `scordelis-lo-{8,16,32,64}x{8,16,32,64}-low-slow-elform10.k` | same as above with ELFORM 10. |
| `scordelis-lo-{8,16,32,64}x{8,16,32,64}-low-slow-elform16.k` | same as above with ELFORM 16. |
| `CHECKS.md` | verification history from the sale project (in Chinese), including the drilling-DOF diagnosis and the full result tables. |

The ELFORM 10/16 decks differ from the ELFORM 2 deck only in the
`*SECTION_SHELL` elform field (1 line). All decks use `*SECTION_SHELL`
ELFORM 2/10/16, NIP 3, and `*HOURGLASS` IHQ = 2, QM = 0.1.

## How to run (LS-DYNA)

```
ls-dyna i=scordelis-lo-16x16-low-slow.k
```

Read the z-displacement of the monitor node from `d3plot` (output every
2 s). For the linear check, use the `*-low-slow.k` decks and the window
**t = 500 s instant / 450–500 s average**, divided by 0.01.

## Expected results (sale, Linux, slow ramp, per-unit, ref 0.3024)

| mesh | ELFORM 2 (inst / avg) | ELFORM 10 (inst / avg) | ELFORM 16 (inst / avg) |
|------|----------------------|------------------------|------------------------|
| 8×8   | 0.36602 / 0.36042 | 0.36602 / 0.36042 | 0.31853 / 0.32069 |
| 16×16 | 0.31528 / 0.31388 | 0.31528 / 0.31388 | 0.29755 / 0.30330 |
| 32×32 | 0.30041 / 0.30280 | 0.30041 / 0.30280 | 0.29736 / 0.30001 |
| 64×64 | 0.29774 / 0.29997 | 0.29774 / 0.29997 | 0.29716 / 0.29929 |

32×32 is within 2% and 16×16 within 5% of the reference; 8×8 (+21%) is
coarse-mesh error. Convergence is monotonic toward the reference.

## Why this case is interesting: the drilling DOF

This benchmark is sensitive to how a shell code treats the **drilling
rotation** (rotation about the shell normal). In sale, the drilling DOF
had zero stiffness, and on this curved shell it leaked into bending
through the angle between adjacent element normals, making the roof
systematically too soft by O(1) — 0.46–0.50 instead of 0.3024 — with no
convergence under mesh refinement. Projecting the nodal rotations onto
the tangent plane fixed it. LS-DYNA explicit has **no drilling control
by default** (the drilling constraint force via `*CONTROL_SHELL`
DRCPSID/DRCPRM is opt-in per part set), so this deck is a genuine test
of what a default DYNA run gives here. If you run it, please report your
numbers — open an issue with the DYNA version, ELFORM, mesh, and the
free-edge midspan displacement.

## Provenance and license

Source: `qashell/09-scordelis-lo` of hao94588/sale (private), verified
2026-11-01. Released under the MIT License (see `LICENSE`).
