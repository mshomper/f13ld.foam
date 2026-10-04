# F13LD.foam

Lightweight, browser-native tool for building periodic Voronoi foams — open-cell strut networks, closed-cell walls, and soap-film Plateau borders — and handing them to [F13LD.mesh](https://mshomper.github.io/f13ld.mesh) for watertight 3MF export. Part of the [F13LD](https://f13ld.app) suite.

Live: https://mshomper.github.io/f13ld.foam

## What it does

- **Seeds** in the cube `[-5, 5]³`: Poisson-disk (spacing derived from the cell count, so coverage is even at any count), Lloyd-relaxed, uniform random, Weaire–Phelan and Kelvin lattices. Periodic mode makes opposite faces match so the cube tiles. 8–500 cells on a log-scaled slider (fine steps at low counts); lattices go down to a single cube (2 seeds for Kelvin, 8 for Weaire–Phelan).
- **Topology**: open (struts on cell edges), closed (walls on cell faces), Plateau (struts with swollen junctions), plus an organic vertex bulge.
- **Exact field** (v0.6.0): the field is the true distance to the nearest cell wall (closed) or cell edge (open, plateau), from the bisector planes of the 8 nearest seeds, searched over the seeds' periodic copies. Walls are 2t thick and struts 2t across everywhere — near nodes, with stretch on, and in tiles of only a few cells. It replaces v0.5.0's gradient normalization (and its toggle), whose struts ran up to a third thicker near nodes.
- **Anisotropy**: stretches the distance metric per axis for elongated, load-aligned cells.
- **Preview**: GPU bake of the exact wall and edge distances per voxel (same construction as the export field, matched to half-float rounding), raymarched with guaranteed-safe step lengths; clip plane, 3×3×3 tile check, mm read-outs. Thickness, plateau k and organic stay live; only seed and stretch changes re-bake.
- **Solid fraction** (v0.4.0): live read-out under the equation, sampled at 48³ from the exact field F13LD.mesh exports and F13LD.lab homogenizes (shown with ≈ for bounded foams, whose faces don't wrap).

## Stiffness estimate (v0.5.0)

**Estimate stiffness** measures the solid fraction of the exact foam (the field F13LD.mesh exports, point-sampled at 64³ in worker threads) and estimates Ex, Ey, Ez, the three shear moduli and Poisson's ratio from laws fitted to 67 F13LD.lab FFT homogenizations of this same geometry (F13LD.lab `docs/FOAM_CALIBRATION.md`):

- open and plateau: E/E_s = 0.724 ρ^1.93 · closed: E/E_s = 0.304 ρ + 0.456 ρ²
- × 0.956 / 0.985 for Poisson-disk seeds (open / closed) · × 1 + 0.118 (1 − e^(−k/0.111)) for plateau borders that add mass
- ν = 0.433 − 0.468 ρ (open), 0.294 (closed) · G = E / 2(1 + ν)
- anisotropy stretch moves stiffness toward the stretched axis as s^2.49 (open) / s^1.63 (closed), keeping the mean

**Exact field (v0.6.0):** the open and plateau laws were fitted on the previous field, whose struts ran thicker near nodes; their estimates carry a note and a wider range until the lab calibration is re-run on the exact field (F13LD.lab `docs/FOAM_CALIBRATION.md` §11). The closed law carries over.

**Calibration status (2026-10-03): provisional.** 27 of the 67 lab runs behind these laws stopped at the lab's old 300-iteration cap (open and plateau foams at 18–35 %, closed at 12–18 % on the coarse grid), and a stopped solve reads stiff, so the open law may read high above ~18 % solid. The re-run at the new 1000-iteration cap, a cell-count pass and a refit are next (F13LD.lab `docs/FOAM_CALIBRATION.md` §8, §10); the constants will move in v0.5.1.

Each value comes with a likely range (fit scatter, seed-to-seed scatter at this cell count, and an open cell-count question still being measured). Ordered lattices (Kelvin, Weaire–Phelan), uniform-random seeds, organic growth, normalize off and densities outside 5–35 % (open) / 12–35 % (closed) get an estimate marked **not calibrated**. The result rides in the exported recipe as a `homogenization` block (F13LD.tpms field names), so F13LD.mesh shows it. For a measured answer, use **Open in F13LD.lab**.

## Handoff

- **Open in F13LD.mesh** and **Open in F13LD.lab** (v0.4.0, stiffness by FFT homogenization) send the recipe after `#r=` in the link. Recipes carry every seed position (up to ~28 KB), and the part after `#` never leaves the browser, so length doesn't matter.
- Only **periodic** foams can be handed to mesh or the lab, or added to the queue; otherwise the tool explains why and offers to turn periodic on.
- **Export JSON** saves the same recipe as a file.

### Recipe

```
{ "family": "foam",
  "meta":     { "tool": "f13ld.foam", "version": "0.5.0", … },
  "domain":   { "world": [-5, 5], "periodic": true, … },
  "seeds":    { "mode", "count", "regularity", "lloyd_iterations", "rng_seed",
                "generator": "FoamSeeds/1", "positions": [x, y, z, …] },
  "anisotropy": { "enabled", "stretch": [sx, sy, sz] },
  "geometry": { "mode": "open|closed|plateau", "thickness", "plateau_k",
                "organic", "normalize", "field": 2, "tile_mm" } }
```

`tile_mm` is the cube edge at the cell size set in the tool; mesh uses it as the default cell size. `field: 2` selects the exact field (v0.6.0+); recipes without it build with the original field in mesh and the lab.

## Shared code

The seed generator (`FoamSeeds`) and the field (`foamSeedsFromRecipe`, `buildFoamSDF`, `buildFoamSDF2`, between the `BEGIN FoamSeeds` / `END foam field` markers in `index.html`) are byte-for-byte copies of F13LD.mesh's `worker/m25-sdf-foam.js` and F13LD.lab's `13d-foam-kernel.js`. Edit all three together (bump `FoamSeeds.VERSION` for generator changes) and run mesh's `tests/foamseeds.js` and the lab's `validate-foam.js`.

## License

MIT
