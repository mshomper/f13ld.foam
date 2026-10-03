# F13LD.foam

Lightweight, browser-native tool for building periodic Voronoi foams — open-cell strut networks, closed-cell walls, and soap-film Plateau borders — and handing them to [F13LD.mesh](https://mshomper.github.io/f13ld.mesh) for watertight 3MF export. Part of the [F13LD](https://f13ld.app) suite.

Live: https://mshomper.github.io/f13ld.foam

## What it does

- **Seeds** in the cube `[-5, 5]³`: Poisson-disk (spacing derived from the cell count, so coverage is even at any count), Lloyd-relaxed, uniform random, Weaire–Phelan and Kelvin lattices. Periodic mode makes opposite faces match so the cube tiles.
- **Topology**: open (struts on cell edges), closed (walls on cell faces), Plateau (struts with swollen junctions), plus an organic vertex bulge.
- **Normalize**: divides the cell-boundary distance by its gradient before subtracting thickness, so walls and struts have one true thickness (2t) everywhere.
- **Anisotropy**: stretches the distance metric per axis for elongated, load-aligned cells.
- **Preview**: GPU bake of the four nearest-seed distances per voxel (grid-accelerated, exact), raymarched with guaranteed-safe step lengths; clip plane, 3×3×3 tile check, mm read-outs.
- **Solid fraction** (v0.4.0): live read-out under the equation, sampled at 48³ from the exact field F13LD.mesh exports and F13LD.lab homogenizes (shown with ≈ for bounded foams, whose faces don't wrap).

## Stiffness estimate (v0.5.0)

**Estimate stiffness** measures the solid fraction of the exact foam (the field F13LD.mesh exports, point-sampled at 64³ in worker threads) and estimates Ex, Ey, Ez, the three shear moduli and Poisson's ratio from laws fitted to 67 F13LD.lab FFT homogenizations of this same geometry (F13LD.lab `docs/FOAM_CALIBRATION.md`):

- open and plateau: E/E_s = 0.724 ρ^1.93 · closed: E/E_s = 0.304 ρ + 0.456 ρ²
- × 0.956 / 0.985 for Poisson-disk seeds (open / closed) · × 1 + 0.118 (1 − e^(−k/0.111)) for plateau borders that add mass
- ν = 0.433 − 0.468 ρ (open), 0.294 (closed) · G = E / 2(1 + ν)
- anisotropy stretch moves stiffness toward the stretched axis as s^2.49 (open) / s^1.63 (closed), keeping the mean

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
                "organic", "normalize", "tile_mm" } }
```

`tile_mm` is the cube edge at the cell size set in the tool; mesh uses it as the default cell size.

## Shared code

The seed generator (`FoamSeeds`, between the `BEGIN FoamSeeds` / `END FoamSeeds` markers in `index.html`) is a byte-for-byte copy of the one in F13LD.mesh's `worker/m25-sdf-foam.js`, so mesh can rebuild the same seeds from the settings if positions are ever missing. Edit both copies together, bump `FoamSeeds.VERSION`, and run mesh's `tests/foamseeds.js`.

## License

MIT
