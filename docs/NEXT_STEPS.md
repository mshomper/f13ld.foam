# F13LD.foam — Next Steps (session handoff)

**As of:** v0.7.0 · 2026-10-05 · exact foam field, v0.6.0 foams, stiffness estimate provisional
**Suite on main:** F13LD.lab v0.17.4 · F13LD.mesh v0.9.4 · F13LD.foam v0.7.0 (after the wet-edge-trim PRs merge)
**Owner direction:** Matt Shomper directs implementation. Analyze and present proposed changes for approval before writing code. Don't over-deliberate.

---

## 0. State

| Version | Date | Contents |
|---|---|---|
| v0.3.0 | — | Periodic Voronoi / Laguerre foams, preview, handoff to F13LD.mesh |
| **v0.4.0** | 2026-10-02 | **Open in F13LD.lab** (`#r=` link); live solid fraction sampled from the exact export field |
| **v0.5.0** | 2026-10-03 | **Estimate stiffness**: Ex/Ey/Ez, shear moduli and ν from laws fitted to 67 F13LD.lab homogenizations, with likely ranges and "not calibrated" notes. The `homogenization` block rides in the recipe (F13LD.mesh shows it) |
| (docs) | 2026-10-03 | Calibration marked provisional: 27 of the lab runs had stopped at the old 300-iteration cap |
| **v0.6.0** | 2026-10-04 | **Exact field** (`geometry.field: 2`): true distance to walls / edges, searched over periodic copies, so walls and struts are 2t everywhere, including stretched foams and single-cube tiles. New foams: wet Plateau borders, fillet / node, two-size mix (power cells, exact volumes), mirror / cubic symmetric seeds, FCC and C15 lattices with disorder, S(k) plot, 8–500 cells. (Built in a separate session from the v0.4.0–v0.5.0 work) |
| **v0.7.0** | 2026-10-05 | **Wet foam edge + continuous field.** The wet field now measures the neighbouring bubbles too, so it no longer jumps across cell faces (those jumps staircased mesh surfaces). New **edge** control (`geometry.edge_min`, world units, default 0.03): cusp tips narrower than it are closed and rounded by a circular blend. Absent / 0 = sharp cusps. Needs F13LD.mesh v0.9.4 / F13LD.lab v0.17.4 for exports to match |

The field code (`FoamSeeds`, `foamSeedsFromRecipe`, `buildFoamSDF`, `buildFoamSDF2`) is byte-identical in three places: here (`index.html`, between `BEGIN FoamSeeds` / `END foam field`), F13LD.mesh `worker/m25-sdf-foam.js`, and F13LD.lab `13d-foam-kernel.js`.

---

## 1. PICK UP HERE

1. **Calibration on the exact field (Matt's GPU, in F13LD.lab).** Follow F13LD.lab `docs/FOAM_CALIBRATION.md` §11 and lab `docs/NEXT_STEPS.md` §1 items 2–3:
   - run the `_field2` run lists at the 1000-iteration cap: first pass 48, plateau 19, cell count 6;
   - load only those results into `foam-fit.html` and export the fit JSON + summary.
2. **v0.6.1: new constants** (propose first, then build):
   - Paste the refit into the `FOAM_CAL` block (`index.html` ~line 2874).
   - Remove the "fitted on the previous field" note and its extra 5 % range on open / plateau estimates (~line 2974), if the refit supports it.
   - Decide on the ±7 % cell-count band, which depends on the cell-count pass.
   - Update the README's "Stiffness estimate" numbers, and replace its "Calibration status" paragraph with the new status.
3. **Decide whether to calibrate the v0.6.0 foams.** Wet, fillet / node, two-size, symmetric seeds, FCC / C15 and disorder all get "not calibrated" estimates today. The lab sweep already takes their columns (`border`, `fillet`, `node`, `size_ratio`, `large_fraction`, `jitter`; topology `wet`), so a pass is a run list plus a fit.
4. **Rectangular tiles, then Z and σ phases** — next feature, plan in §3. Direction agreed (2026-10-04); three decisions still open before building (§3.4).

## 2. Exporting foams through F13LD.mesh

F13LD.mesh v0.9.3 makes weld groups with foam much faster:
- solids are exported exactly as imported;
- only the space around the foam is meshed;
- foam fillets skip a 7× calculation;
- the work is spread across all cores.

Mesh's own follow-ups are in its `docs/SESSION_RECAP_2026-10-04.md`. Two of them touch foam:
- **Open edges after simplify.** Mesh simplification leaves some open edges on very fragmented foam (thin walls at coarse quality). This predates v0.9.3.
- **Triangle-count estimate.** The export panel's triangle estimate is far too low for foam (about 5k shown, 395k actual in the test scene).

## 3. Rectangular tiles + Z and σ phases (next feature)

### 3.1 Why rectangular (orthorhombic) only

A box with its own length on each axis covers every non-cubic need:
- **slender parts** — tiles long in one direction;
- **σ phase** — tetragonal unit cell (c/a ≈ 0.52);
- **Z phase** — hexagonal cell, which fits a rectangular cell with sides 1 : √3 (twice the seeds).

Skewed (triclinic) boxes would touch far more code for no extra reach.

### 3.2 Plan (propose to Matt before building)

1. **Recipe.** `domain.size: [Lx, Ly, Lz]` in tile units, centred on the origin. No `size` = the 10-unit cube, so every existing recipe builds exactly as before.
2. **Shared code (all three copies, `FoamSeeds.VERSION` 3).**
   - Period per axis everywhere: seed generators, `powerCells`, the field's wrap and padded copies.
   - Mean spacing from the box volume.
   - Lattices fill whole cells per axis.
3. **F13LD.foam.**
   - Box proportion controls; lattices set their own proportions.
   - Non-cubic bake texture and raymarch box.
   - mm read-outs per axis; `tile_mm` per axis.
   - Add **Z** and **σ** to the lattice list, with the disorder slider.
4. **F13LD.mesh.** Cell size per axis, so one tile maps to a rectangular brick. Tiling, shape clipping and the export-size estimate follow.
5. **F13LD.lab.** The FFT solver only takes cubic samples: refuse non-cubic foams with an explanation, as it refuses non-periodic ones today (unless §3.4 decides otherwise).
6. **Stiffness estimate.** Mark non-cubic tiles "not calibrated".
7. **Tests.** Cube recipes byte-identical in mesh `harness.js`. Add lab `validate-foam.js` cases: brute-force reference in a rectangular box, tiling per axis, cell face counts for Z and σ.

### 3.3 Z and σ seed positions

- **Source.** Take positions from the crystallographic tables (σ: P4₂/mnm, 30 sites per cell; Z: P6/mmm, 7 per hexagonal cell → 14 in the 1 : √3 rectangular cell).
- **Check.** Verify by counting Voronoi faces per cell before shipping: Frank–Kasper cells have 12, 14, 15 or 16 faces. This is how C15 was checked (16 and 12).

### 3.4 Open decisions (Matt)

- Go ahead with rectangular tiles as described?
- Z and σ in the same pass, or tiles first and the lattices later?
- Lab: refuse non-cubic foams for now, or look at rectangular cells in its solver?

## 4. Later ideas (from the 2026-10-04 research)

Shipped in v0.6.0: exact field, two-size Laguerre cells, hyperuniform Lloyd + S(k), lattice disorder, FCC / C15, symmetric seeds, wet borders, fillet / node. Still open, roughly in value order:

1. **Explicit periodic cell graph** (vertices / edges / faces, built once from the seeds). It unlocks:
   - short-strut cleanup at a mm threshold;
   - struts that taper toward the nodes;
   - read-outs for faces per cell, edge lengths and minimum strut length in mm;
   - **Delaunay struts** (seed to seed, stretch-dominated) and a Voronoi + Delaunay hybrid.
2. **Plateau-law relaxation** of the graph: 109.47° / 120° node angles, a light stand-in for Surface Evolver.
3. **Sheet ("soft") foams.** A smooth minimum of the exact plane distances gives bicontinuous stochastic sheets. Intersecting two such fields from different seeds is the stochastic analogue of PI-TPMS pipe networks.
4. **Inverse design.** Nudge seeds toward a target stretch or stiffness, trained on F13LD.lab runs, via Synth.

Full write-up with sources: [`RESEARCH_2026-10-04.md`](RESEARCH_2026-10-04.md). Decisions and findings from the v0.6.0 session: [`SESSION_RECAP_2026-10-04.md`](SESSION_RECAP_2026-10-04.md).

## 5. Rules for changes

- **Shared field code:** edit all three copies together; bump `FoamSeeds.VERSION` for generator changes. Then run:
  - F13LD.mesh `tests/foamseeds.js <mesh build> <this index.html>`;
  - F13LD.lab `node validate-foam.js`;
  - mesh `tests/harness.js` (open-cube exports must stay byte-identical for unchanged families).
- **Older recipes:** recipes without `geometry.field` must keep building with the original field everywhere.
- **Versions:** bump the version in `index.html` (header and recipe `meta.version`) and the README on each release.
- **Commits and PRs:** no claude.ai session links; `Co-Authored-By: Claude …` trailers are fine. GitHub GraphQL is blocked from Claude sessions, so open and merge PRs with `gh api` (REST).
- **Wording:** never use the words "genuine" / "genuinely" in docs or UI text.
- **Branches:** all merged branches can go (Matt, 2026-10-04). Claude sessions can't delete them, so Matt does, or turns on "Automatically delete head branches" in Settings.
