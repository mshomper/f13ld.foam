# F13LD.foam — UI refresh log

Branch `ui-refresh`. Audit: [`UI_AUDIT.md`](UI_AUDIT.md).

| Stage | Date | Commit | Checks | Open decisions |
|---|---|---|---|---|
| 1. Audit | 2026-11-25 | none yet | screenshots 1440 / 390; no page errors; no horizontal scroll at 390 / 700 / 1024 / 1440; 52 controls mapped (grep + runtime count); help (i) 17; compiles per tick measured (thickness, plateau k, border recompile every tick) | D1 accent (sand, chroma), D2 re-bake timing, D3 bake resolution control, D4 sections, D5 anisotropy toggle, D6 missing estimate (i), D7 `meta.version`, D8 cell-size wording |
| 2. Layout shell | 2026-11-25 | see `git log` (`UI refresh stage 2`) | 0 page errors at 390 / 700 / 1024 / 1440; no horizontal scroll (page and control column) in all 4 sections at 4 widths; text below 11 px: 0; interactive targets under 44 px: 0 of ~430 checked per width (page + ranges, counting `::after` hit boxes) except the brand link (24 px, shared chrome, as TPMS) and the help drawer's Tutorial link (engine); export JSON identical to `main` in 9 of 9 states (default; open + fillet + node; closed + Lloyd 80 cells; wet border + edge; Kelvin + disorder; two-size mix; Weaire–Phelan + anisotropy; Poisson rng 7 + periodic + anisotropy; with homogenization) apart from `meta.version` (now `0.8.1`, D7) and results cleared on design change; help (i) 18 (was 17); 23 of 23 help cards have a working "show" at 1440 and 390 (phases: plateau, wet, two-size, Kelvin so each hidden row is visible); screenshots `docs/ui-audit/stage2-{desktop,phone}-{seeds,structure,stretch,analyze}.png` (+ `-peek` on phone) | Desktop rail is a plain-text fallback until stage 3 mounts the liquid rail. Export button sits at the bottom of Analyze (header Recipe menu is stage 3). Quality Low/Med/High left as is (stage 4 / D3). Shared F13LD-VIEW block still draws the glyph `◐ view` (byte-identical suite block). Stiffness glyph canvas buffer is 180 px (set while hidden): looks soft. Brand link and Tutorial link are under 44 px (shared chrome). Help anchors for S(k) and Estimate moved to `#skTitle` and `#estLabel` |

## Decisions (Matt, 2026-11-25)

Shared, all wave 1 tools:
- **Quality:** automatic, as in TPMS. Remove the manual quality control (buttons or slider); its help (i) goes with it.
- **Stale analysis:** clear homogenization / stiffness results when the design changes, and on import. Exports carry results only for the current design.
- **Missing help cards:** add them; the (i) count rises by that number (record the new count).
- **Field sliders that re-bake:** show a low-res preview bake while dragging, full bake on release. Keep it only if it measures faster to first change than today and does not slow the full bake.
- **Accent:** derived from the tool colour in the F13LD brand guidelines v2.0 (hue kept, lifted to L 0.80 for the dark UI). Final value pending Matt.

This tool:
- **Accent:** sand `#E8D4A2` = `oklch(0.874 0.069 89)` (not in the brand guide). Chroma question (D1) still open.
- **Bake resolution (D3):** fixed 192³ (today's High); `domain.bake_res` exports 192. Older recipes with 128/160 import with no warning (view setting).
- **Re-bake timing (D2):** preview bake (96³) during the drag, full bake on release.
- **Missing estimate (i) (D6):** add it; 17 → 18.

- D1 keep sand exactly; judge in preview. D4 sections as proposed. D5 keep the anisotropy switch. D7 `meta.version` matches the app version. D8 fix the cell-size help text.

Stage 1 approved by Matt 2026-11-25 ("use your picks").
