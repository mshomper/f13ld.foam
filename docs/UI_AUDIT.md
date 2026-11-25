# F13LD.foam — UI audit (stage 1)

Branch `ui-refresh` · 2026-11-25 · read-only audit, no code changed.
Brief: `f13ld-ui-refresh-handoff.md`. Template: `liquid-ui/templates/f13ld/`. Reference: F13LD.tpms v1.4.0.
No `CLAUDE.md` in this repo. Repo rules from `docs/NEXT_STEPS.md` §5 apply (shared field code byte-identical with mesh/lab, never "genuine", version bump in header + recipe `meta.version` + README).

Screenshots: `docs/ui-audit/`. Headless Chrome 1440×900 and 390×844, swiftshader.

| File | What |
|---|---|
| `desktop-1440.png` | first screen, plateau foam |
| `desktop-1440-full.png` | whole page |
| `desktop-1440-wet.png` | wet topology (border / edge rows) |
| `desktop-1440-bimodal-full.png` | two-size mix, periodic on, S(k) plot |
| `desktop-1440-estimate-full.png` | stiffness estimate results |
| `desktop-view-menu.png` | shared view menu open |
| `desktop-help.png` | help drawer |
| `phone-390.png`, `phone-390-full.png` | phone first screen, whole page |
| `phone-390-lattice-full.png` | Kelvin lattice (disorder row) |
| `phone-390-help.png` | help drawer on phone |

---

## 1. Current state

- **Version** v0.8.1 (header line 242, script banner line 468). Recipe `meta.version` is still `'0.7.0'` (line 2916).
- **File** single `index.html`, 3,845 lines, 207 KB.
- **Blocks**
  | Lines | Block |
  |---|---|
  | 10–213 | `<style>` app CSS (hex palette, `--accent:#E8D4A2`) |
  | 214–465 | markup: header, `#cw` viewer + overlays, `#panel` (one long column) |
  | 466–3425 | app `<script>`: F13LD-SHADE (477–527), F13LD-VIEW (528–594), state, FoamSeeds (shared, ~671–1262), seed worker, S(k), GPU bake (1744–2185), live shader (2187–2420), camera, overlays, UI wiring (2725–2897), export (2898–2999), render loop (3017–3053), stiffness estimate (3056–3415) |
  | 3436–3543 | help drop-in styles `#f13h-theme`, `#f13h-style` |
  | 3545–3798 | help drop-in `script#f13h-script` |
  | 3801–3843 | queue drop-in (`F13LDQueue.mountButton`, `alert()` fallback) |
- **Drop-ins:** help (yes), queue (yes), view (yes, F13LD-VIEW v1, byte-identical suite block). No liquid-ui, no recipe import.
- **Renderer.** WebGL2. Seeds on CPU (worker for two-size / symmetric modes) → GPU bake of exact wall / edge distances into an RGBA16F 3D texture (128/160/192³, one draw per z-layer, sync `readPixels`) → full-screen ray-march of the baked field. Second WebGL2 context for the stiffness glyph (`#ellcv`). Canvas: CSS width × min(DPR, 1.5), height min(width, 420 px). `preserveDrawingBuffer:true` (queue snapshot), default `antialias`.
- **Export JSON** (`buildExportJSON`, line 2903):
  ```
  family:'foam'
  meta{tool:'f13ld.foam', version:'0.7.0', timestamp, preset}
  domain{world:[-5,5], periodic, bake_res}
  seeds{mode, count, count_actual, regularity, min_spacing, lloyd_iterations, size_ratio,
        large_fraction, jitter, rng_seed, generator:'FoamSeeds/2', positions[3N], weights?}
  anisotropy{enabled, stretch:[x,y,z]}
  geometry{mode, thickness, plateau_k, organic:0, fillet, node, border, edge_min,
           normalize:true, field:2, tile_mm}
  homogenization: null | {grid, E_solid_GPa, poisson, volume_fraction, Ex..Gyz_GPa, nu_eff,
                          anisotropy, method, calibrated, uncertainty_log10_sd, notes}
  ```
  **`meta.tool` = `'f13ld.foam'` — confirmed.** Also used by the queue drop-in (line 3824). Lab and Mesh route on it. Do not rename. `family:'foam'` is an existing key (since v0.3.0, Mesh routes on it); keep it, add nothing.

## 2. Findings

| # | Issue | Where | Sev | Rule |
|---|---|---|---|---|
| 1 | Thickness, plateau k and border are baked into the shader as literals: every tick recompiles. Measured 10 links / 10 ticks (sThick), 5/5 (sPlateauK), 5/5 (sBorder), ~33 ms per tick in swiftshader | `shaderChange` 2741–2749; `buildFrag` 2229 (`T`, `extraExpr` 2205, `featureT` 2212) | high | L1 |
| 2 | Fillet, node, edge, border and X/Y/Z stretch re-bake the whole 3D texture on each pause > 30 ms in a drag; seed sliders (cells, regularity, Lloyd, ratio, share, disorder, RNG) regenerate seeds + bake after 60 ms. Bake measured 4.4–16 s in swiftshader (one blocking GPU job) | `bakeChange` 2746, `anisoChange` 2782, `scheduleBake` 2180, `scheduleRegen` 1731, `runBake` 2115 | high | L1, L2, respond < 400 ms |
| 3 | Render loop is continuous: `frame()` always re-requests; auto-rotate on by default; glyph context redrawn every frame (0.003 rad per frame, rate-dependent) | 3017–3053, 3359 | high | L2, render-quality §4 |
| 4 | Quality Low/Med/High recompiles (step count compiled in) and changes `domain.bake_res` in the export | 610–615, 2806–2811, 2923 | med | L2, export identity |
| 5 | No `webglcontextlost` handler. In headless swiftshader the first page in a fresh browser lost its context during the first bake (blank viewer, `CONTEXT_LOST_WEBGL`); later pages were fine. A long bake on a weak GPU can do the same | 596, `runBake` 2115 | med | robustness |
| 6 | One long page: viewer, field read-out and all four groups stacked; no sections | 254–463 | high | L6, one focus |
| 7 | Hit targets far below 44 px: help (i) 14, ranges 16 tall, number inputs 19, overlay buttons 20–21, topology 23, header buttons 25, Export 30 | CSS 81–175; measured | high | 44 px |
| 8 | Text below 11 px: 9, 10, 10.5 px in 12 rules; 122 visible text elements < 11 px | CSS 10–213 | med | type scale |
| 9 | Symbol glyphs as icons: `⬡ Mesh`, `▦ Lab`, `⊕ Queue`, `↻ reseed`, `⬢` section bullets, `⚠`, `×`, `◐ view` (shared block) | 240–241, 276, 278, 349, 400, 412, 424, 3813; 553 | med | drawn icons |
| 10 | Hex palette, inline colours (`#a78bfa`, `#110e07`, `#0c0a06`, rgba glows); no OKLCH tokens; `box-shadow` glow on `#eqbar` | 15–36, 106, 192, 206, 444–446 | med | OKLCH, L8 |
| 11 | Help (i) for "Estimate stiffness" never renders: anchor `{row:'#runBtn'}` finds no `.sc-group` or `<label>`. 18 anchors, 17 placed | 3650–3655, `rowLabel` 3754 | low | L7 |
| 12 | Cell size is labelled "display only, does not affect the JSON export", but it sets `geometry.tile_mm` | 304 title, help card `cellsize`, 2962 | low | wording |
| 13 | `meta.version` `'0.7.0'` lags header v0.8.1 (NEXT_STEPS §5 says bump both) | 2916 | low | repo rule |
| 14 | Mesh-warning banner, export "Copied ✓" text swap and queue `alert()` fallback: three ad-hoc feedback paths | 275–279, 3009–3010, 3831 | low | shared header message |
| 15 | Anisotropy on/off and periodic on/off are separate toggles. Anisotropy off = stretch 1,1,1 visually, but `anisotropy.enabled` is exported | 350, 413, 2815, 2827 | low | fewer toggles |
| 16 | S(k) box draws an empty 100 px area with "periodic seeds only" when periodic is off | 403–406, 1695 | low | chunking |
| 17 | Lloyd slider max changes with mode (60 / 12); count slider is log-mapped position 0–1000, not a count | 2833–2836, 2758–2779 | info | widget fit (see §4) |
| 18 | No horizontal scroll at 390 / 700 / 1024 / 1440 (scrollWidth = clientWidth) | measured | pass | no h-scroll |
| 19 | No page errors at any width. Only console errors are the blocked external font / queue requests in the harness | measured | pass | — |

## 3. Proposed section map

Workflow from the help intro: place seeds → pick how boundaries become solid → stretch → measure. One section at a time; rail on desktop, tab bar on phone. Icons are kit drawn icons.

| Section (icon) | Group | Controls (visible at once) |
|---|---|---|
| **Seeds** (`cell`) | Layout | seed mode dropdown · periodic toggle · cells slider · cell size (mm) |
| | Spread (by mode) | regularity · Lloyd iter · size ratio · large share · disorder · RNG seed + reseed (`refresh`) — max 6 shown (two-size mix) |
| | Check | S(k) plot; one line instead of the empty box when periodic is off |
| **Structure** (`volume`) | Field | foam field read-out (`#eqbar`, solid %) |
| | Shape | topology segmented (open / closed / plateau / wet) · thickness or border · plateau k or edge · fillet · node — max 5 |
| **Stretch** (`tune`) | Anisotropy | anisotropy toggle · X · Y · Z (axis tints) |
| **Analyze** (`analyze`) | Estimate | E_s · ν_s · Estimate stiffness (progress button) · status |
| | Results | glyph, metric grid, likely-range bells, mm row, method note; rail/tab badge when new or stale |

- **Header:** Recipe menu (`recipe`): Open recipe… (`open`), Export JSON (`download`), Queue (`queue`), Choose queue…, Mesh (`layers`), Lab (`flask`). Replaces the Mesh / Lab header buttons and the Export JSON button.
- **Viewer overlays:** seeds toggle, tile 3× toggle, clip radio X/Y/Z, view menu, bake badge. Quality strip: see decision D3.
- **Mesh warning** (non-periodic hand-off) → `F13UI.message(…, {action:{label:'Turn periodic on'}})`.

## 4. Control → widget map

Inventory by grep (`<select`, `type="range"`, `type="number"`, `checkbox`, `<button`, `createElement('button')`) and by runtime count at 1440 (1 select, 15 ranges, 4 numbers, 5 checkboxes, 43 buttons = 21 static + view menu 2 + help trigger/close 2 + help (i) 17 + queue 1).

| # | Control | Type | Widget | Notes |
|---|---|---|---|---|
| 1 | `#seedMode` | select (3 optgroups, 10 options) | dropdown `w-seedmode` | keep optgroups as group headers; `change` handler auto-sets periodic for lattice / heavy modes |
| 2 | `#sCount` | range 0–1000 (log position) | slider, `format: pos→count`, `parse: count→pos` | shows cell count, not position; hint `#vsCount_mm`; mapping min changes with mode (`countMin`) |
| 3 | `#sThick` | range | slider | hint `#vsThick_mm`; uniform after stage 4 |
| 4 | `#sPlateauK` | range | slider | plateau only |
| 5 | `#sBorder` | range | slider | wet only; hint `#vsBorder_mm` |
| 6 | `#sEdge` | range, step 0.005 | slider, 3 decimals | wet only; two hints (`#vsEdge_mm`, `#vsEdge_warn`, orange) |
| 7 | `#sFillet` | range | slider | hint `#vsFillet_mm` |
| 8 | `#sNode` | range | slider | hint `#vsNode_mm` |
| 9 | `#sSpacing` | range | slider (label "regularity") | hint `#vsSpacing_mm` |
| 10 | `#sLloyd` | range, max 60 or 12 | slider | **max changes at runtime**: kit slider has no `setRange`; kit fix or remount (risk R3) |
| 11 | `#sRatio` | range | slider | two-size only |
| 12 | `#sLarge` | range | slider | two-size only |
| 13 | `#sJitter` | range | slider (label "disorder") | lattices only |
| 14 | `#sAnisoX` | range | slider, accent `axisX` | rows are `.aniso-axis` with `span.axis-lbl`, no `<label>`: restructure to `.sc-group.sl` rows; `setDisabled` when anisotropy off |
| 15 | `#sAnisoY` | range | slider, accent `axisY` | same |
| 16 | `#sAnisoZ` | range | slider, accent `axisZ` | same |
| 17 | `#cellSizeMM` | number | native, restyled 44 px | free decimal entry; no kit number widget |
| 18 | `#rngSeed` | number | native, restyled 44 px | integer entry; paired with reseed |
| 19 | `#Es` | number | native, restyled 44 px | as TPMS Analyze |
| 20 | `#nuS` | number | native, restyled 44 px | as TPMS Analyze |
| 21–25 | view menu checkboxes ×5 (Shadows, Occlusion, Warm/cool, Shade cut faces, Lime edges) | checkbox (F13LD-VIEW) | toggles, `CONFIG.viewMenu:true` | shared block, wired by template |
| 26–29 | `#mOpen` `#mClosed` `#mPlateau` `#mWet` | button group | segmented `w-topo` | `syncOn:['class']` (native uses `.active`) |
| 30–32 | `#btnQLow` `#btnQMed` `#btnQHigh` | button group | segmented `sm` in overlay, or removed | see D3; recompiles today |
| 33–35 | `#clipPillX` `#clipPillY` `#clipPillZ` | button group | radio `CONFIG.clip` (axis accents, allowNone) | `get:()=>clipState.axis`; app needs an `F13UI.onClipChange` call in the pill handler |
| 36 | `#btnShowSeeds` | toggle button (`.active`) | toggle `sm` in overlay | add `role=switch` / `aria-checked` to native in stage 2 so `CONFIG.switches` can watch it |
| 37 | `#btnTile` | toggle button (1× / 3×) | toggle `sm` "tile 3×" | same |
| 38 | `#periodicToggle` | toggle button (`.on`, text) | toggle `w-periodic` | same; text "periodic: on/off" becomes the label |
| 39 | `#anisoToggle` | toggle button (`.on`, text) | toggle `w-aniso` | same; see D5 |
| 40 | `#reseedBtn` | action | native, 44 px, `refresh` icon | one-shot action, no kit widget needed |
| 41 | `#runBtn` | action (async) | progressButton `CONFIG.run` | `runEstimate` is async and toggles `disabled`; add `F13UI.onAnalyze('running'/'new'/'stale')` calls (`estStale` 3199) |
| 42 | `#exportBtn` | action | hidden native under Recipe menu | `CONFIG.recipe.exportBtn:'exportBtn'` |
| 43 | Mesh header button (`onclick=openInMesh()`) | action | Recipe menu item | `CONFIG.recipe.mesh` |
| 44 | Lab header button (`onclick=openInLab()`) | action | Recipe menu item | `CONFIG.recipe.lab` |
| 45 | `#meshWarnFix` | action | header message action | replace banner |
| 46 | `#meshWarnClose` | action | header message dismiss | replace banner |
| 47 | view menu button (`.f13v>button`) | menu | native (shared block) | suite-wide byte-identical block; its `◐` glyph is a suite issue |
| 48 | view menu reset (`.f13v-r`) | action | native (shared block) | template resyncs toggles after it |
| 49 | Queue main (`.f13ldq-main`, from queue client) | action | Recipe menu "Queue" | client mounts it; don't change its API |
| 50 | Queue caret (`.f13ldq-caret`) | menu | Recipe menu "Choose queue…" | same |
| 51 | help trigger `.f13h-trigger` | action | native (help system) | restyle 44 px; drawn icon instead of `?` text |
| 52 | help close `.f13h-close` | action | native (help system) | `close` icon |
| — | help (i) `.f13h-i` ×17 | action | native, unchanged | count must stay 17 (or 18 if #11 is fixed, D6) |

**Total: 52 controls** (1 select, 15 ranges, 4 numbers, 5 checkboxes, 27 buttons including 4 segmented / radio groups' members and 4 toggles), plus 17 help (i). Grep cross-check: 1 + 15 + 4 + 5 + 21 static buttons + 6 generated (view 2, queue 2, help 2) = 52. The queue offline fallback (`#f13ldq-fallback`) replaces items 49–50 when the client fails to load.

Widgets: 15 sliders, 1 dropdown, 2 segmented (+1 if quality stays), 1 radio, 4 + 5 toggles, 1 progress button, 1 Recipe menu, 1 sections pair. Native kept: 4 number fields, reseed, shared view-menu button + reset, help buttons.

## 5. Render cost

- **Shader build.** `buildFrag()` (2229) emits GLSL with literals for: topology (`baseExpr` d.x / d.y), wet flag, plateau k (`extraExpr`), thickness `T`, feature size (`featureT`: thickness or 0.3 × border), step count (quality). `marchConsts` → uniforms `uMarchM`, `uInvLip` (already per frame). Field distances come from the baked 3D texture.
- **`rebuildShader()` call sites (2406):**
  | Line | Caller | Fires per input tick? |
  |---|---|---|
  | 2743 | `sThick` via `shaderChange` | **yes** — 10/10 ticks measured |
  | 2744 | `sPlateauK` via `shaderChange` | **yes** — 5/5 |
  | 2749 | `sBorder` (`scheduleBake` + `shaderChange`) | **yes** — 5/5, plus a re-bake |
  | 2793 | topology buttons via `shaderChange` | no — click; structural (keep) |
  | 2809 | quality buttons | no — click; becomes unnecessary |
  | 3421 | startup | once |
- **Re-bake per tick** (not a compile, but heavier): `sFillet`, `sNode`, `sEdge`, `sBorder`, `sAnisoX/Y/Z`, anisotropy toggle, wet ↔ other topology. **Regen + bake**: `sCount`, `sSpacing`, `sLloyd`, `sRatio`, `sLarge`, `sJitter`, `rngSeed`, reseed, periodic, seed mode.
- **Tiers today:** Low 128³ / 192 steps, Med 160³ / 256 (default), High 192³ / 384. Steps compiled in; bake resolution also exported as `domain.bake_res`.
- **Loop:** continuous rAF, never sleeps; auto-rotate on; glyph context drawn every frame while results show. `f13View.interacting` already drops shadows while a range is dragged.
- **Changes for render-quality.md:**
  1. Thickness, plateau k, feature size and step limit → uniforms (fixed loop bound 384 with early `break`). Topology / wet stay compile keys; cache programs by source string; count compiles (`drawStats.compile`), assert 0 per drag.
  2. Context `antialias:false` (needed for `blitFramebuffer`); keep `preserveDrawingBuffer` for the queue snapshot. Fixed canvas size, DPR ≤ 2 (today 1.5) — check bake/frame cost.
  3. Interactive tier (~160 steps, adaptive 0.4–0.85 offscreen, LINEAR blit) during orbit, zoom, clip scrub, any input, auto-rotate; rest frame (384 steps, shadows) in strips after 150 ms idle.
  4. On-demand loop (`requestRender` / `kick`); glyph on change, ≤ 30 fps spin, paused while main view moves, Reduce Motion respected.
  5. **Bake (foam-specific, not in render-quality.md):** decouple bake resolution from ray steps; bake on slider release (`change`) or after ~250 ms idle, not on each 30 ms pause; keep the last bake on screen meanwhile. See D2.
  6. Add `webglcontextlost` / `restored` handling (re-create programs, re-bake).

## 6. Recipe import

- **Recognise:** `meta.tool === 'f13ld.foam'`, or structure `family:'foam'` + `seeds.positions` + `geometry.mode`. Foreign `meta.tool` values (`f13ld.tpms`, `noise-scaffold-explorer`, `grain`, `beam`, `beam-builder`, …) → message naming the tool, abort.
- **Version keys:** `meta.version` (`0.3.0`…`0.7.0`), `seeds.generator` (`FoamSeeds/1`, `/2`), `geometry.field` (absent = original field, `2` = exact), `geometry.edge_min` (absent before 0.7.0 = 0, sharp), `geometry.organic` (> 0 only before 0.6.0), `geometry.normalize` (false possible before 0.6.0).
- **Migrations / warnings:** no `field` → foam now always builds the exact field: warn "rebuilt with the exact field; solid fraction changes". `organic > 0` → not available, warn, ignore (fillet 0). Missing `edge_min` in wet → 0. Older generator → use the stored positions (do not regenerate).
- **Apply order** (no setter API exists; state lives in `state` + native inputs + handlers):
  1. quality / bake res (D3) → 2. seed mode (`#seedMode` change; it may force periodic on) → 3. periodic (`setPeriodic`) → 4. count (write `state.count` + `syncCountSlider()`: the log slider cannot hit every count, e.g. ~2-cell steps near 500) → 5. regularity, Lloyd (cap depends on mode), size ratio, large share, disorder, RNG seed → 6. topology (`.mode-btn` click → `syncTopologyUI`) → 7. thickness, plateau k, border, edge, fillet, node → 8. anisotropy enabled + stretch → 9. cell size from `tile_mm` (`cellSizeMM = tile_mm / 10 × meanSpacing(count_actual)`; stored per browser) → 10. seeds: `applySeeds({seeds: positions, weights, minSpacing: min_spacing})` without `scheduleRegen` (heavy modes would otherwise go to the async worker) → one `rebuildShader()` + one `scheduleBake()`.
- **Setters available:** `setPeriodic`, `syncTopologyUI`, `syncSeedModeUI`, `syncCountSlider`, `applySeeds`, `scheduleRegen`, `scheduleBake`, `rebuildShader`, `updateEq`, `updateMMHints`, `onDesignChange`, `setToggle`. Missing: set anisotropy (toggle only flips), set quality, set clip.
- **Exclude from restore / round-trip compare:** `homogenization` (estimate; call `estStale`), `meta.timestamp`. Derived, compare but don't set: `meta.preset`, `seeds.count_actual`, `seeds.min_spacing` (restored via `applySeeds`), `geometry.tile_mm` (via cell size).
- **Round-trip risk:** deep-equal needs positions loaded verbatim; regenerating from settings would only match for `FoamSeeds/2` and non-worker modes. `bake_res` must round-trip (D3).
- **Undo:** snapshot `buildExportJSON()` before apply; re-apply on "Undo".

## 7. Help system

- 23 cards in 7 sections (Topology, Geometry, Seed distribution, Anisotropy, Viewport, Stiffness, Handoff). 18 have anchors; **17 (i) buttons render** (desktop and phone); `estimate` anchor fails (finding 11).
- `ROW_SEL = '.sc-group'`. Anchors that must survive: `{row}` → `#mOpen`, `#sCount`, `#cellSizeMM`, `#sThick`, `#sPlateauK`, `#sBorder`, `#sEdge`, `#sFillet`, `#sNode`, `#sSpacing`, `#sLloyd`, `#sRatio`, `#sJitter`, `#rngSeed`, `#runBtn` (each needs a `.sc-group` ancestor with a `<label>`); `{into}` → `#seedLabel`, `#sk-wrap`, `#anisoLabel`.
- Card ids `f13h-card-<id>`, classes `.f13h-i`, `.f13h-trigger`, `.f13h-drawer`, `.f13h-scrim`; trigger inserted first in `.fh-right`.
- Stage 3 adds `helpTargets` ("show" → switch section, scroll, flash) for the 17 anchored cards.

## 8. Accent colour

| Token | Hex | OKLCH |
|---|---|---|
| `--accent` (sand) | `#E8D4A2` | 0.874 0.069 88.8 |
| `--accent-mid` | `#C9B27E` | 0.771 0.074 86.8 |
| `--accent-deep` | `#8B6F3E` | 0.558 0.075 80.2 |
| `--accent-dark` | `#3a2a14` | 0.298 0.042 73.8 |
| `--accent-soft` | `#1a1408` | 0.195 0.024 84.0 |
| header foam icon stroke | `#5C4624` | 0.410 0.058 76.9 |
| foam surface (shader base) | `vec3(0.910,0.831,0.635)` = sand | — |

**Proposed: sand, `oklch(0.874 0.069 88.8)`. To confirm with Matt.** Sand's chroma (0.069) is about half the kit palette (0.14–0.19): beads, flood and glow will read pale next to other tools. Options in D1. Kit `amber` (0.84 0.16 75) is close in hue; the sections "stale" badge uses amber, so on foam it should use coral or a distinct hue.

## 9. Risks

- **R1 Bake cost dominates.** Uniforms fix compiles, but fillet / node / edge / border / stretch drags still need a 3D re-bake (seconds on weak GPUs). Without a bake-on-release rule the UI stays slow.
- **R2 `bake_res` in the export.** Adaptive quality removes Low/Med/High; if bake resolution silently follows it, exports change and break byte-identity with `main`.
- **R3 Kit gaps.** Slider needs `setRange` (Lloyd max 60 ↔ 12); no number-field widget (4 natives stay); toggle wiring expects `role=switch` / `aria-checked` the foam buttons don't have yet.
- **R4 Context loss.** No handler today; seen once in headless runs on a cold browser. The verifier must warm up the browser or the first screenshot is blank.
- **R5 Shared code.** FoamSeeds / field blocks must stay byte-identical with Mesh `m25-sdf-foam.js` and Lab `13d-foam-kernel.js`; F13LD-SHADE / F13LD-VIEW with every tool. Stage 2–4 edits must stay outside those markers.
- **R6 Async paths.** Heavy seed modes return from a worker; `runEstimate` is async. Recipe import, progress button and round-trip tests must wait on them.
- **R7 Swiftshader timing.** Bake 4–16 s, compile ~33 ms per tick headless; compare ratios against `main` only.

## Decisions for Matt

- **D1 Accent.** Keep sand exactly (`oklch(0.874 0.069 88.8)`), or lift its chroma a little (e.g. `oklch(0.86 0.10 85)`) so the liquid beads and flood don't look grey?
- **D2 Re-bake timing.** Fillet, node, edge, border and stretch need a full re-bake. Re-bake when the slider is released (fast, preview lags during the drag), or keep live re-bakes during the drag (slow)?
- **D3 Quality buttons.** Ray-march quality goes adaptive. The bake resolution (128/160/192, written to the export as `bake_res`) still needs a home: keep a small 3-way "bake" switch on the viewer, move it to the view menu, or fix it at 160?
- **D4 Sections.** Seeds → Structure → Stretch → Analyze, with the Recipe menu in the header replacing the Mesh / Lab / Export buttons. OK, or merge Stretch into Structure?
- **D5 Anisotropy toggle.** Keep the on/off switch (the export has `anisotropy.enabled`), or drop it and treat 1 / 1 / 1 as off (changes the export)?
- **D6 Missing help (i).** The "Estimate stiffness" help button has never shown. Fix it (count goes 17 → 18) or leave the count at 17?
- **D7 Recipe version.** Exports say `meta.version 0.7.0` while the app is v0.8.1. Bump it with this release, or leave it as the recipe-format version?
- **D8 Cell size wording.** The help says cell size doesn't touch the export, but it sets `tile_mm`. Fix the text in this pass?
