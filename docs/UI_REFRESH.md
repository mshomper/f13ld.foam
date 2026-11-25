# F13LD.foam — UI refresh log

Branch `ui-refresh`. Audit: [`UI_AUDIT.md`](UI_AUDIT.md).

| Stage | Date | Commit | Checks | Open decisions |
|---|---|---|---|---|
| 1. Audit | 2026-11-25 | none yet | screenshots 1440 / 390; no page errors; no horizontal scroll at 390 / 700 / 1024 / 1440; 52 controls mapped (grep + runtime count); help (i) 17; compiles per tick measured (thickness, plateau k, border recompile every tick) | D1 accent (sand, chroma), D2 re-bake timing, D3 bake resolution control, D4 sections, D5 anisotropy toggle, D6 missing estimate (i), D7 `meta.version`, D8 cell-size wording |

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
