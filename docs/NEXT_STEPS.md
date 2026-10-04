# F13LD.foam — Next Steps (session handoff)

**As of:** v0.6.0 · 2026-10-04 · exact foam field, v0.6.0 foams, stiffness estimate provisional
**Suite on main:** F13LD.lab v0.17.3 · F13LD.mesh v0.9.3 · F13LD.foam v0.6.0
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

## 2. Exporting foams through F13LD.mesh

F13LD.mesh v0.9.3 makes weld groups with foam much faster:
- solids are exported exactly as imported;
- only the space around the foam is meshed;
- foam fillets skip a 7× calculation;
- the work is spread across all cores.

Mesh's own follow-ups are in its `docs/SESSION_RECAP_2026-10-04.md`. Two of them touch foam:
- **Open edges after simplify.** Mesh simplification leaves some open edges on very fragmented foam (thin walls at coarse quality). This predates v0.9.3.
- **Triangle-count estimate.** The export panel's triangle estimate is far too low for foam (about 5k shown, 395k actual in the test scene).

## 3. Rules for changes

- **Shared field code:** edit all three copies together; bump `FoamSeeds.VERSION` for generator changes. Then run:
  - F13LD.mesh `tests/foamseeds.js <mesh build> <this index.html>`;
  - F13LD.lab `node validate-foam.js`;
  - mesh `tests/harness.js` (open-cube exports must stay byte-identical for unchanged families).
- **Older recipes:** recipes without `geometry.field` must keep building with the original field everywhere.
- **Versions:** bump the version in `index.html` (header and recipe `meta.version`) and the README on each release.
- **Commits and PRs:** no claude.ai session links; `Co-Authored-By: Claude …` trailers are fine. GitHub GraphQL is blocked from Claude sessions, so open and merge PRs with `gh api` (REST).
- **Wording:** never use the words "genuine" / "genuinely" in docs or UI text.
- **Branches:** all merged branches can go (Matt, 2026-10-04). Claude sessions can't delete them, so Matt does, or turns on "Automatically delete head branches" in Settings.
