# F13LD.foam — Session recap 2026-10-04 (v0.6.0)

Research, the exact field and the v0.6.0 foams, shipped with F13LD.mesh v0.9.2 and F13LD.lab v0.17.3. Foam PR #5, mesh PR #8 and lab PR #13 were all merged on 2026-10-04. Plan going forward: [`NEXT_STEPS.md`](NEXT_STEPS.md).

## Decisions (Matt)

- **Order of work:** exact field first, then new foams.
- **Wet foam:** a fourth topology, alongside open / closed / plateau.
- **Organic:** removed for new designs, replaced by **fillet** plus a **node** size control. Old recipes with organic still build as before.
- **Cell sizes:** a **two-size mix**, chosen over a single spread control.
- **Symmetric seeds:** both **mirror** (orthotropic) and **cubic**.
- **Cell count:** **8–500**, since 1,024 was too dense to be practical. Lattices go down to a single cube.
- **Calibration:** hold the queued lab calibration runs until the exact field merged. Generators updated to write `_field2` run lists.
- **Non-cubic tiles:** fine as long as they tessellate. Rectangular tiles + Z / σ are the next feature (NEXT_STEPS §3).

## Measured findings

- **Strut thickness on the previous field.**
  - Struts read **8 % thick on average, up to a third thicker** near nodes (worst 5 %).
  - Walls were within about 2 %.
  - At the same thickness, open foams were 13–17 % more solid than the exact field.
  - The open-strut field also had small jumps, up to about 12 % of thickness, where the third-nearest cell swaps.
- **Previous field at low cell counts.** It compared each seed only once, by its nearest copy, so with few cells it missed a cell's walls with its own copy. A one-cube Kelvin tile read 5.3 % solid instead of 6.5 %.
- **Exact field accuracy.** Matches a brute-force reference over every periodic copy to about 1e-15 (lab `validate-foam.js` §8). The GPU bake matches the JS field to half-float rounding in every mode.
- **Speed.**
  - Per point, the exact field costs 1.0–1.8× the previous one at 20–500 cells.
  - The GPU bake took 1.1–1.4× as long under software GL.
  - Page startup is unchanged, about 5 s on Matt's phone with old and new builds alike.
- **Calibration thicknesses.** At the same target solid fraction, open and plateau runs need about 7 % more thickness on the exact field (A-open-8: 0.229 → 0.246). Closed runs barely move.
- **Hyperuniformity (S(k) read-out).** 60 Lloyd rounds bring S at long wavelengths to about 0.01. Jittered FCC stays about 0.05, which fits perturbed lattices being hyperuniform.

## Gotchas found while building

- **GPU single precision in symmetric layouts.** Cubic-symmetric seeds made exact ties, and float32 accepted a line or vertex nearer than its own planes. The bake now:
  - rejects any line or vertex nearer than its own planes;
  - treats near-parallel planes (det < 1e-6) as parallel.
  The JS field is unaffected.
- **Symmetric Lloyd and near-duplicates.** Lloyd rounds pull seeds onto mirror planes, leaving near-duplicate pairs (sliver cells). Relaxed seeds within 0.2 × spacing of a mirror are snapped onto it.
- **Two-size weight solve.**
  - After a centroid step a small cell can empty out. Newton can't start from that, so the solve restarts from zero weights.
  - Intermediate rounds solve to 2 %, the last to 0.2 %. Seeds take 0.02–3.3 s for 8–500 cells, so they run in a worker, and bakes wait for pending seeds.
- **Power-cell clipping on perfect lattices.** Cut planes through existing corners need a tolerant inside / on / outside test, or corners get dropped.
- **GLSL:** `out` is a reserved word.
- **Testing headless:** Playwright needs `--no-proxy-server`, and external fonts must be blocked or page load hangs.
