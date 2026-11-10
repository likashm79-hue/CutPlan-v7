# CutPlan v0.7.0

CutPlan is a browser-only cutting-list and board-nesting tool for furniture, cabinet, wardrobe and panel workshops. This release upgrades the working v0.6.6 project in place: the parser, optimizer, guillotine validator and familiar browser-only workflow are retained. No application framework or backend was introduced, and the original v0.5.0 project remains included under `backup-v0.5.0/`.

## v0.7.0 highlights

- **Unified parsed-row table:** every parsed cutting-list row stays in one continuous table. Each row displays its 0.8 mm and 2 mm edge-banding counts (`L` and `W`) alongside the reference, dimensions, quantity and rotation setting. The table is not clipped to a short scroll window. Use the row's **Edit** action to open the edge-banding editor directly on that reference; changes update the visible row immediately.
- **Waste/offcut drawing:** leftover rectangular regions remain highlighted in amber in both the live layout and Print / Save PDF. No `W01` / `W02` text or other labels are printed inside waste regions.
- **A4 PDF layout:** each board is printed on its own A4 portrait page. The board drawing fills nearly the full available page height. Small-part markers that cannot fit legibly inside parts are explained in a compact schedule to the right of the drawing, rather than beneath it.
- **Board heading:** the current board name is centered above the drawing and printed in bold.
- **Color-preserving print:** the print stylesheet explicitly preserves the amber waste highlights, including when the browser print renderer is not instructed to print generic page backgrounds.

## Historical v0.6.4 highlights

- Dedicated full-viewport Edge Banding editor opened from the cutting-list toolbar; close button and Escape return to the list. The reference selector and all four edge-count fields remain visible together without sideways scrolling.
- Rebuilt the cutting-list table as a compact fixed-layout grid so it fits narrow screens without horizontal swiping. On narrow screens, inputs reflow into labelled row cards. (v0.7.0 restores a compact edge-banding summary and row-level edit action.)
- Kept the dedicated Edge Banding editor with part/reference selector and separate 0.8 mm / 2 mm material groups; v0.7.0 exposes each row's saved counts in the main cutting-list table.
- Added a Full screen workbench button for larger editing space, with a fixed-overlay fallback when the browser Fullscreen API is unavailable.
- Board drawings highlight every terminal rectangular offcut in amber without drawing labels inside those regions. Global and per-board counts distinguish reusable offcuts from small scrap rectangles. v0.7.0 preserves these highlights in PDF output. Margins and kerf are not counted as offcut boxes.
- Added fragment-aware placement scores and offcut-preservation scores to the existing deterministic portfolio. When board count and utilization tie, candidate comparison prefers lower small-scrap fragmentation and larger reusable offcuts. Stock board count remains the first priority; no extra board is accepted just to improve remainder shape.
- Existing CSV `_` / `~` marker imports, manual per-part values and edge-banding totals remain supported.

### Board naming

- Set **Board name / label prefix** before optimizing. For example, `Bedroom wardrobe panel` generates `Bedroom wardrobe panel 01`, `02`, etc.
- Rename each board in its result card. The display name is shown in the card, SVG drawing and printed PDF.
- Board IDs are deterministic internal IDs derived from the board's part placement signature, not from the display name.
- Individual overrides are keyed by board ID and stored in browser `localStorage` where the page is served from a normal browser origin. Changing the prefix does not replace a custom board name. An identical rerun re-applies the override while the board ID remains the same.
- Display names never enter the optimizer and do not change geometry.

### Optimizer

- Three effort levels: **Quick** (12 portfolio candidates), **Standard** (64), and **Deep** (192). The current Standard/Deep portfolios preserve the earlier 48/144 candidate set and add 16/48 fragment- and offcut-aware candidates. Multi-board plans additionally try up to four measured board-elimination probes, with a hard cap of one fewer board than the current best.
- Diversified deterministic ordering: area, longest side, shortest side, perimeter, aspect ratio, width, length, difficult-to-place, rotation-aware, mixed and input order, plus seeded shuffled orders.
- Alternative fit/split scores: short-side, long-side, area, balanced split, waste, offcut preservation, fragment reduction and largest-box preservation.
- Seeded deterministic tie-breaking. The seed and winning strategy are shown in the result statistics.
- A valid initial solution is retained and progress callbacks publish the best validated solution as the search continues. The browser uses `nesting-worker.js` when a same-origin worker can be created, and falls back to the same engine on the main thread if Worker setup is unavailable.
- The final board-elimination phase repacks the full part list under an `N-1` board limit. It cannot accept an extra board merely for a small utilization gain.
- Actual search statistics include candidates attempted, valid candidates, improvements, elapsed search time, winning strategy, reusable rectangular offcut area/count, small-scrap count, largest reusable rectangle and stop reason.
- Results remain heuristic; no global optimum is claimed. There is no OR-Tools dependency or arbitrary non-guillotine rectangle-packing final stage.

### Guillotine geometry and validation

- Each placement is produced by recursive rectangular free-region splits. Cuts span the full parent region and create kerf-separated rectangular child regions.
- Each board retains a hierarchical `cutTree` with its split sequence.
- The independent validator reconstructs the tree and checks parent/child geometry, cut span, kerf gap, terminal part leaves, unique piece identity, rotation restrictions, bounds, overlaps and part count. A deliberately corrupted child-region test is rejected.
- Kerf and margin are included in the geometry calculations. Rotation-aware edge-banding metadata and existing parser semantics are retained.

### Drawings and print

- Label placement measures text with the actual drawing font in the browser canvas and evaluates it at the board's print scale. It tries several text line arrangements and positions and reduces the font gradually down to 6 pt for in-box labels.
- If reference plus dimensions cannot fit readably, a marker is placed inside the part and a matching external schedule lists marker, reference, dimensions and quantity. No part label is silently discarded.
- Monochrome portrait drawings retain solid board and part boundaries only. No coloured edge-banding graphics, dashed cut-guide overlays or decorative cut lines are drawn.
- Dimensions use `mm` suffixes, e.g. `780 mm`; there are no Length/Width headings.
- Browser print uses A4 portrait, one board per page, a centered bold board name, a near-full-height drawing and a compact small-part schedule beside the board.

## Run on Ubuntu / Linux

Node.js is required for tests. A modern browser is required for the UI. From a terminal:

```bash
cd /path/to/CutPlan-v0.7.0
python3 -m http.server 8000
```

Open `http://localhost:8000` in Chromium, Chrome or Firefox. Keep `index.html`, `cutting-list-parser.js`, `nesting-engine.js`, `drawing-layout.js` and `nesting-worker.js` together in the same folder. For normal use, serve the folder over localhost rather than opening the HTML as a `file://` URL; that gives browser storage and same-origin Worker support.

The optimizer, parser, drawing layout and board naming work locally. Internet access is only used for the optional Tesseract.js OCR and PDF.js import libraries used by scanned-image/PDF extraction.

## Tests and benchmarks

Run the retained original regression suite and the new tests:

```bash
node backup-v0.5.0/test-engine.js
node test-engine.js
node test-drawings.js
node benchmark-engine.js
node compare-v062-v063.js deep
node compare-v062-v063.js standard
python3 test-v070-ui.py
```

`test-engine.js` covers the original parser and edge-banding behaviour, width/length/optimal modes, deterministic seeds, all effort levels, fixed and permitted rotation, kerf values, difficult/narrow/repeated/mixed datasets, candidate metrics, async progress and hierarchical cut-tree tamper rejection. `test-drawings.js` covers small and narrow rectangles, long references, marker containment and marker-to-schedule mapping. `test-v070-ui.py` (requires Python Playwright and Chromium at `/usr/bin/chromium`) verifies the unified parsed-row/banding table, row-specific banding edits, mobile overflow, unlabeled amber waste regions, side callouts, centered bold board title and PDF print layout. `benchmark-engine.js` runs the preserved v0.5.0 engine and current engine; `compare-v062-v063.js` compares offcut quality using identical Deep or Standard settings against the bundled v0.6.2 engine fixture. Results are in `BENCHMARK-RESULTS.csv`, `COMPARISON-V062-V063-deep.csv`, `COMPARISON-V062-V063-standard.csv` and `BENCHMARK-REPORT.md`.

See `V0.7.0-VERIFICATION.md` for the current v0.7.0 regression and PDF inspection results. `TEST-REPORT.md` and `STATUS.md` retain historical v0.6.x verification notes. See `STATUS.md` for verification limitations and remaining work.

## Project files

| File | Purpose |
|---|---|
| `index.html` | Single-page UI, board labels, SVG drawing generation, print styles and worker integration |
| `cutting-list-parser.js` | Preserved v0.5.0 cutting-list parser and recovery behaviour |
| `nesting-engine.js` | Deterministic optimization portfolio, guillotine cut-tree generation and validator |
| `drawing-layout.js` | Print-scale-aware in-box label fitting and marker placement |
| `nesting-worker.js` | Dedicated worker entry point for non-blocking optimization |
| `test-engine.js` | Engine/parser regression and adversarial geometry tests |
| `test-drawings.js` | Drawing text-fit and marker/schedule tests |
| `test-v070-ui.py` | v0.7.0 Chromium UI, print CSS and PDF export checks |
| `benchmark-engine.js` | Side-by-side v0.5.0 and current v0.6.4 benchmark runner |
| `BENCHMARK-RESULTS.csv` | Captured v0.5.0 / v0.6.4 benchmark output |
| `compare-v062-v063.js` | Reproducible offcut comparison in Deep or Standard effort |
| `benchmark-fixtures/v0.6.2-nesting-engine.js` | Frozen v0.6.2 engine for apples-to-apples comparison |
| `COMPARISON-V062-V063-deep.csv` / `...-standard.csv` | Offcut-quality measurements at both search efforts |
| `TEST-REPORT.md` | Baseline, unit, browser and PDF checks |
| `BENCHMARK-REPORT.md` | Apples-to-apples benchmark analysis |
| `STATUS.md` | Delivery status and honest verification limits |
| `backup-v0.5.0/` | Untouched original five project files |

## Workshop caution

Always confirm sheet dimensions, usable margin, measured blade kerf, grain direction, saw sequence, clamping, edge-banding specification and material before cutting. Validate the plan against the actual saw and production workflow. A blank kerf is a provisional zero-kerf assumption, not a production-ready setting.

## v0.7.0 — current release

The cutting-list table is now the single source of truth for parsed rows and banding review. Each row displays `0.8 mm: Lx / Wx` and `2 mm: Lx / Wx`, and its **Edit** button opens the guided banding editor on that exact reference. The complete list remains in one continuous view rather than a clipped table window.

For each board's PDF page, the board name is centered and bold at the top, the board drawing fills nearly the full page height, and small-part marker explanations appear in a narrow right-hand column. Leftover rectangles remain amber in print/PDF and have no labels printed inside them. One board is printed per A4 portrait page.

Run on Ubuntu / Linux:

```bash
cd /path/to/CutPlan-v0.7.0
python3 -m http.server 8000 --bind 127.0.0.1
```

Open `http://127.0.0.1:8000` and hard-refresh with `Ctrl+Shift+R`. The header must show **v0.7.0**.

Run the current regression and browser/PDF checks:

```bash
node backup-v0.5.0/test-engine.js
node test-engine.js
node test-drawings.js
python3 test-v070-ui.py
```

The browser test generates `verification/v0.7.0-print-check.pdf` for print inspection. The checked release includes its verification summary in `V0.7.0-VERIFICATION.md`.
