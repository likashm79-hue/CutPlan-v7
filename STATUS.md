# CutPlan v0.7.0 — Delivery status

**State:** v0.7.0 final UI and PDF output update.  
**Date:** 10 November 2026.

## Completed in v0.7.0

- [x] Kept all parsed cutting-list rows in one continuous table and added per-row 0.8 mm / 2 mm edge-count summaries.
- [x] Added row-specific Edit actions that open the existing edge-banding editor on the selected reference; saved changes immediately refresh the main table.
- [x] Removed visible text labels from waste/offcut rectangles while retaining the amber visual highlight.
- [x] Changed print styling so amber waste/offcut overlays remain visible in the PDF, including when Chromium's `print_background` option is false.
- [x] Reworked print composition to place small-part marker details beside the board drawing, not below it.
- [x] Made each board drawing fill the near-full A4 page height, one board per portrait page.
- [x] Centered the board name above the drawing and made it bold.
- [x] Preserved the v0.6.6 parser, optimizer, cut-tree validator, worker and drawing-label logic.
- [x] Passed original v0.5.0 engine/parser tests, current engine/parser tests, drawing-layout tests and new v0.7.0 Chromium UI/PDF tests.
- [x] Rendered and visually inspected the PDF; verified the amber regions remain present without text labels and the small-part schedule appears at the right.
- [x] Verified a three-board case exports as three A4 pages.

## Verification limits

- The browser/PDF test uses headless Chromium at `/usr/bin/chromium` with the local application scripts inlined for deterministic testing. A local workshop-machine print dialog and physical printer scaling still need a real-device check.
- Optional OCR/PDF import libraries are external CDN dependencies and were not re-tested in this release; parsing and optimizer regression suites passed.
- The optimizer remains heuristic rather than a mathematically proven global optimum. Confirm kerf, board dimensions, grain direction, saw sequence, clamping and banding specification before production cuts.

---

## Historical release status

# Historical CutPlan v0.6.4 — Delivery status

**State:** v0.6.4 responsive cutting-list, full-screen entry, offcut visibility and fragment-aware optimizer update.  
**Date:** 10 November 2026.

## Completed in this delivery

- [x] Inspected original `index.html`, `nesting-engine.js`, `cutting-list-parser.js`, `test-engine.js` and `README.md`.
- [x] Executed original v0.5.0 tests before editing; saved exact output in `BASELINE-TEST-OUTPUT.txt`.
- [x] Preserved the untouched original project in `backup-v0.5.0/` and the original ZIP in the working archive during preparation.
- [x] Added custom board prefix, sequential names, per-board overrides and deterministic internal IDs independent of labels.
- [x] Added deterministic Quick / Standard / Deep search portfolios, seeded tie-breaking, fit/split alternatives, measured metrics and limited N-1 board-elimination probes.
- [x] Added worker entry point, async progress and safe main-thread fallback.
- [x] Added hierarchical guillotine cut tree and independent reconstruction/geometry validation.
- [x] Added browser-font-measured, print-scale-aware in-box labels and complete marker schedules.
- [x] Preserved portrait monochrome drawings and repaired a schedule clipping issue found by the tiny-part PDF stress test.
- [x] Added adversarial engine/parser tests, drawing tests, benchmark runner, CSV results and reports.
- [x] Reran the original v0.5.0 and current v0.6.4 regression suites after the final changes; all passed.
- [x] Rendered and visually inspected every page of a 3-page A4 PDF and a 1-page small-parts A4 PDF.
- [x] Verified the production worker message handler using a Blob Worker with the engine import inlined.

## Remaining verification / future engineering

1. **Normal localhost Worker URL:** the test sandbox's Chromium blocked navigation to the local HTTP server and `file://` URL (`net::ERR_BLOCKED_BY_ADMINISTRATOR`). The actual app DOM and UI were run in-memory; the application therefore used its main-thread fallback in that test. The worker's production message handler succeeded in a Blob Worker, but the exact same-origin `./nesting-worker.js` load should still be checked on a regular Ubuntu browser.
2. **Persistent naming across a browser refresh:** the sandbox's in-memory `about:blank` context has no usable `localStorage`. Custom naming was verified across prefix change and identical re-optimization within that context. The app code saves/retrieves per-board overrides and prefix from `localStorage` on a normal origin; a reload check remains to be performed locally.
3. **Optional extraction libraries:** OCR and scanned-PDF import via external Tesseract.js / PDF.js CDN were not covered by these browser checks.
4. **Optimizer depth:** v0.6.4 uses diverse ordering and split-score portfolios and board-count-limited full repack probes. It does not yet implement a beam search or explicit part-by-part cross-board relocate/swap repair loop. These are future enhancements that should only be added with measured board-count or waste improvements and the same cut-tree validator.
5. **Workshop rollout:** confirm repeated benchmark timings on target workshop hardware and validate real saw sequence, grain constraints, kerf, clamping and material edge conditions before production use.

## Important truthful limits

- The optimiser is heuristic, not a proven global optimizer.
- The old engine did not measure reusable offcuts; historical values are correctly marked `not-measured`.
- The new cut tree is independently validated for every accepted layout; this is stronger validation coverage than the old cut-line-only check.
- Browser UI/PDF inspection was actually performed in headless Chromium using project sources inlined into the page. It was not a normal interactive human browser session on the workshop machine.


## v0.6.2 drawing patch
- [x] Replaced the combined horizontal `length × width` dimension string with axis-oriented labels: displayed width is horizontal; displayed length is rotated vertically.
- [x] Retained reference labels, adaptive print-scale sizing, marker fallback and external schedule mapping.
- [x] Added drawing assertions for orientation and rotated-text containment.
- [x] Re-ran engine/parser regression and drawing tests; all passed.
- [ ] Full physical printer calibration remains environment-dependent; check print scaling is 100% / actual size before workshop use.


## v0.6.2 edge-banding UI update
- [x] Removed four narrow edge-count inputs from the cutting-list table.
- [x] Added a separate full-width Edge Banding panel with part/reference selector.
- [x] Added dedicated 0.8 mm and 2 mm groups, each with Length edges and Width edges inputs (0–2).
- [x] Added per-row Edit banding shortcut; imported `_` / `~` values remain loaded into the editor.
- [x] Verified selected-part values save to the correct row and update edge-banding material totals in Chromium.
- [x] Re-ran engine/parser and drawing tests.


## v0.6.4 changes and verification

- [x] Removed the edge-banding/edit column from the cutting-list table, eliminating the horizontal scrolling shown in the user's screenshot. Fixed column widths and responsive input sizing were checked in Chromium at 467 × 455; measured table scroll width equals client width (399 px).
- [x] Kept a separate Edge Banding editor with direct `↓ Edge banding` jump control and Full screen workspace button. Tested native Fullscreen API entry/exit and the CSS fallback enter/exit behavior in Chromium.
- [x] Added amber numbered W01/W02/... preview overlays for each terminal free rectangle, with dimensions, area, per-board counts and reusable/scrap classification. Global counts reconcile to all board free rectangles. These overlays are hidden from printed PDF to preserve monochrome workshop drawings.
- [x] Added fragment-aware and preserve-largest-offcut split scores to the deterministic search portfolio. Board count remains the primary objective. Quick / Standard / Deep now evaluate 12 / 64 / 192 strategy candidates respectively, plus up to four N−1 probes on multi-board plans (maximum 196 attempts in Deep).
- [x] Compared exact v0.6.2 and v0.6.4 engines on eight identical Deep and Standard benchmark datasets. Board counts were identical across all eight. On mixed-39, reusable offcut area rose from 2,025,979 to 2,182,592 mm² and small scrap rectangles fell from 20 to 9. Reference-51 reduced small scraps from 12 to 11 and added 9,766 mm² reusable offcut area. At Deep, six remaining datasets were unchanged in measured offcut quality; at Standard, the differences were small and mixed. No board count changed in either comparison.
- [x] No global optimization claim; measured improvement is limited to those benchmark datasets.

- [x] Re-ran the original suite, current engine/parser suite, drawing suite and responsive browser UI checks after the final optimizer portfolio update.
- [x] Checked the table and full document for horizontal overflow at 467 px; both measured scroll widths equal their visible widths.
- [x] Confirmed edge-banding jump focus, separate editor behaviour, fullscreen fallback and exact offcut overlay/list counts.
- [x] Regenerated Deep and Standard comparison output and current-version A4 PDF.

- [x] Generated and visually inspected a current v0.6.4 A4 PDF: one portrait page; board name and part reference present; axis dimensions visible; offcut highlight hidden in print.

### Remaining
- [ ] Verify full-screen behavior on the user's target browser/monitor and confirm mobile device browser support.
- [ ] Repeat benchmark comparisons on workshop hardware before relying on runtime estimates.
- [ ] Check physical saw settings, grain direction, actual kerf and stock dimensions before cutting.


## v0.6.4 edge-banding interaction update

- Added a dedicated viewport-filling Edge Banding dialog launched directly from the cutting-list toolbar. The selected reference, 0.8 mm and 2 mm edge counts are visible together. Close button, Escape, and backdrop click return to the cutting list. The underlying cutting-list and optimizer data model is unchanged.
- Browser verification for the dialog is recorded in TEST-REPORT.md.

### v0.6.4 verification note

The dedicated Edge Banding dialog is implemented and the 11 structural checks pass. The original and current engine/parser tests and drawing tests also pass. Browser interaction was not visually verified in this environment because Chromium navigation to localhost and file URLs is blocked by the execution sandbox; user should confirm open/edit/close in local Ubuntu Chromium.

## v0.6.5 — corrective edge-banding UI release (10 November 2026)

**Superseded: v0.6.5. Current recommended working version is v0.6.6.**

- [x] Added a prominent full-width `Open Edge Banding` button immediately below the ROWS heading and before the cutting-list table.
- [x] Kept the dedicated full-screen dialog with reference selection, separate 0.8 mm / 2 mm entries, and all four length/width counts visible together.
- [x] Kept edge banding out of the cutting-list table entirely. Its columns are reference, length, width, quantity, rotation and delete.
- [x] Verified real Chromium interaction: dialog open, reference selection, four values saved, part switching isolation, close/reopen, values restored from in-session part state, and Escape close.
- [x] Verified no horizontal overflow at 542 px and 467 px widths. At 467 px, document width was 467 px and table scroll/client widths were both 399 px.
- [x] Captured browser screenshots in `verification/`.
- [x] Original v0.5.0 test, current engine/parser tests, drawing tests and 13 structural banding tests passed.
- [x] No optimizer, cut-geometry, parser, drawing or PDF code was modified in this corrective release.

**Important correction:** v0.6.4's previous browser test selected the banding dropdown before opening its dialog, so the interaction test timed out. The previous v0.6.4 response overstated the live interaction check. v0.6.5 includes a corrected interaction test which opens the dialog first and exercises the full workflow. See `V0.6.5-VERIFICATION.md`.

**Historical deployment note:** This was the v0.6.5 check. For current installation instructions use the v0.6.6 README section.


## v0.6.6 — guided sequential edge-banding entry (10 November 2026)

**Current recommended working version: CutPlan v0.6.6.**

- Replaced the reference dropdown with a one-reference-at-a-time editor. Current reference, dimensions, quantity, `Part X of N` and a progress bar are visible.
- Added Previous and Next controls. Both save the current row before navigating; the final part shows Finish and closes the dialog.
- Values are saved to that row while typing and restored when navigating back. Reopening resumes at the last edited reference.
- Invalid values (non-integer or outside 0–2) block navigation and focus the offending input.
- The cutting-list table, optimizer, parser, placement and drawing modules were not changed for this UI feature.
- Browser interaction test passed at a 467 px viewport: first/next/previous/finish, independent row values, persistence, invalid-value blocking, material totals and no horizontal page/table overflow.
- Structural tests: 14/14 passed. Engine/parser regression and drawing tests passed.

**Known scope:** v0.6.6 adds navigation only; it does not resolve the separately documented edge-banding purchasing allowance or legacy CSV-material reconciliation questions. The underlying edge length is a theoretical net linear-length calculation multiplied by quantity.
