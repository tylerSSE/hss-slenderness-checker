# Agent instructions — HSS Slenderness Checker

Read this before making any changes to this app.

## What this app is

A public-facing engineering tool by Southern Steel Engineers. It checks whether an HSS column wall is non-slender per AISC 360-22 Table B4.1a, which determines whether a standard shear tab connection is acceptable. See `README.md` for the product summary and `../HSS-column-slenderness.md` for the engineering reference (the `hss-slenderness-check` skill).

## Who uses it

Internal SSE engineers *and* external users via the SSE website (currently beta). Outside engineers will make real connection-design decisions from this output. Wrong numbers here become wrong connections in the field.

## Non-negotiable rules

1. **The skill (`HSS-column-slenderness.md`) is the engineering source of truth.** Every calc change must be consistent with it. If you believe the skill is wrong, stop and confirm with the user — don't silently diverge. When changes require it, update the skill in the same session.

2. **Verify against the worked examples.** Any edit touching `tdes`, `Fy`, `b`, `h`, `D`, `λ`, or `λr` must preserve (values follow the AISC Manual rounding convention — see rule 5):
   - HSS8x8x1/4, A500-C, square → tdes = 0.233, λ = 31.3, λr = 33.7 → **NON-SLENDER**
   - HSS8x8x3/16, A500-C, square → tdes = 0.174, λ = 43.0, λr = 33.7 → **SLENDER**
   - HSS10.000x0.375, A500-C, round → tdes = 0.349, λ = 28.7, λr = 63.8 → **NON-SLENDER**
   - HSS16.000x0.250, A500-C, round → tdes = 0.233, λ = 68.7, λr = 63.8 → **SLENDER**

   And the `λr` quick-reference values: A500-B rect 35.2, A500-B round 69.3, A500-C rect 33.7, A500-C round 63.8, A1085 rect 33.7, A1085 round 63.8. Current A500 Fy values (AISC Manual 16th Ed., Table 2-4): Gr. B = 46 ksi rect/round, Gr. C = 50 ksi rect/round — do *not* revert to the legacy lower round values (42 / 46).

3. **Don't guess.** If a shape-table entry, Fy, tdes factor, or formula is uncertain, ask or check the AISC source rather than inferring. Structural calcs can't be vibe-coded.

4. **Keep the disclaimer visible.** The on-screen and report disclaimers stating that qualified engineers must independently verify results must stay. If changing the phrasing, keep the substance.

5. **Preserve the AISC Manual rounding convention.** The app rounds `tdes = tnom × tdes_factor` to 3 decimals (round-half-up, `Math.round(x*1000)/1000`) *before* using it in `b = B − 3·tdes`, `h = H − 3·tdes`, or `λ`. Displayed `tdes` is 3 dp, `b`/`h` are 3 dp, and `λ`/`λr` are 1 dp (3 significant figures in the typical 10–99 range). This matches the values published in AISC Manual Tables 1-11 and 1-12 and lets engineers cross-check the app against the manual line-for-line. Do not revert to full-precision tdes or to 2-dp λ/λr display.

6. **Minimum HSS size.** Shape tables (`SQUARE_HSS`, `RECT_HSS`, `ROUND_HSS`) deliberately exclude any section whose smallest dimension is less than 3 in. These shapes are out of scope for the tool — do not add them back without an explicit request.

## Source of truth

- Canonical: `src/App.jsx`
- Ignore: the root-level `../hss-slenderness-checker (4).jsx` (stale artifact from the original Claude artifact prototype — do not edit).

## Code layout notes

`src/App.jsx` is a single ~1700-line file. Roughly:
- Top: `STEEL_GRADES` constants (tdes factor, Fy for rect vs round).
- Shape tables: `SQUARE_HSS`, `RECT_HSS`, `ROUND_HSS` arrays.
- `SectionGraphic` component (SVG of the section).
- `HSSSlendernessChecker` default export (the app — state, UI, save/print modes).

Refactoring the file is fine when it helps, but prefer minimal targeted edits for calc bugs. If extracting data tables or components, do it as its own change, not bundled with a calc fix.

## Change workflow

1. Read the relevant section of `../HSS-column-slenderness.md` before touching calc code.
2. Make the smallest edit that fixes the issue.
3. Mentally (or with a quick script) re-run the worked examples and spot-check the λr table.
4. If the skill needs updating to match, update it in the same change.
5. Note in the response which skill invariants were re-verified.

## UI / UX changes

UI polish is welcome but:
- Don't remove or visually de-emphasize the disclaimer, input units, or the pass/fail result.
- Keep the print layout (`printMode` = `'single'` and `'project'`) working — users generate PDFs from these.
- Preserve the shape/grade/face selectors' behavior; `connectionFace` only applies to rectangular.

## Out of scope unless asked

- Adding connection types beyond shear tabs (the skill is scoped to shear-tab suitability).
- Building alternative-connection design logic (through-plates, stiffened tabs, etc.) — the app only flags when standard shear tabs don't work.
- Account systems, server-side storage, or analytics.

## Support contact

Public users are directed to support@southernsteelengineers.com for bugs and feature requests.
