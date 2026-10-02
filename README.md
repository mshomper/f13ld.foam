# F13LD.foam

Lightweight, browser-native tool for building periodic Voronoi foams — open-cell strut networks, closed-cell walls, and soap-film Plateau borders — and handing them to [F13LD.mesh](https://mshomper.github.io/f13ld.mesh) for watertight 3MF export. Part of the [F13LD](https://f13ld.app) suite.

Live: https://mshomper.github.io/f13ld.foam

## What it does

- **Seeds** in the cube `[-5, 5]³`: Poisson-disk (spacing derived from the cell count, so coverage is even at any count), Lloyd-relaxed, uniform random, Weaire–Phelan and Kelvin lattices. Periodic mode makes opposite faces match so the cube tiles.
- **Topology**: open (struts on cell edges), closed (walls on cell faces), Plateau (struts with swollen junctions), plus an organic vertex bulge.
- **Normalize**: divides the cell-boundary distance by its gradient before subtracting thickness, so walls and struts have one true thickness (2t) everywhere.
- **Anisotropy**: stretches the distance metric per axis for elongated, load-aligned cells.
- **Preview**: GPU bake of the four nearest-seed distances per voxel (grid-accelerated, exact), raymarched with guaranteed-safe step lengths; clip plane, 3×3×3 tile check, mm read-outs.

## Handoff

- **Open in F13LD.mesh** sends the recipe after `#r=` in the link. Recipes carry every seed position (up to ~28 KB), and the part after `#` never leaves the browser, so length doesn't matter.
- Only **periodic** foams can be handed to mesh or added to the queue; otherwise the tool explains why and offers to turn periodic on.
- **Export JSON** saves the same recipe as a file.

### Recipe

```
{ "family": "foam",
  "meta":     { "tool": "f13ld.foam", "version": "0.3.0", … },
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
