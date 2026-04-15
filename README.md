# HSS Slenderness Checker

A web tool by [Southern Steel Engineers](https://southernsteelengineers.com) that checks whether an HSS column wall is **non-slender** per AISC 360-22 Table B4.1a — the threshold that determines whether a standard bolted single-plate (shear tab) connection is acceptable for a given HSS section.

**Status: Beta.** Feedback, bugs, and feature requests: [support@southernsteelengineers.com](mailto:support@southernsteelengineers.com).

---

## What it does

Given a shape, steel grade, and (for rectangular sections) the face the shear tab attaches to, the app:

1. Applies the design-thickness factor (`tdes = tnom × 0.93` for A500 B/C; `1.00` for A1085).
2. Picks the correct `Fy` for the grade and shape type.
3. Computes the slenderness ratio `λ` (`b/tdes`, `h/tdes`, or `D/tdes` depending on shape and face).
4. Computes the limiting ratio `λr` (`1.40·√(E/Fy)` for rectangular/square; `0.11·(E/Fy)` for round).
5. Reports **NON-SLENDER** (pass — standard shear tab OK) or **SLENDER** (fail — alternative connection required).

Users can save multiple checks to a project, view section graphics, and print per-check or full-project reports.

## Audience

- **Internal:** SSE engineers and detailers as a quick check during connection design.
- **External:** Clients, GCs, and outside engineers via the SSE website.

Because outside engineers will act on the output, correctness is non-negotiable. The app carries a disclaimer that all results must be independently verified by a qualified engineer before use in design.

## Engineering reference

The canonical engineering reference is `HSS-column-slenderness.md` in the workspace root (the `hss-slenderness-check` skill). **Code and skill must stay in sync.** If a bug fix changes calc behavior, update the skill too; if the skill is wrong, update both.

Worked examples from the skill that should always pass as regression checks:
- HSS8x8x1/4, A500 Gr. C, square → `λ = 31.41`, `λr = 33.72` → **NON-SLENDER**
- HSS8x8x3/16, A500 Gr. C, square → `λ = 42.76`, `λr = 33.72` → **SLENDER**

## Stack

- Vite + React (single-file app in `src/App.jsx`)
- Deployed on Vercel

## Development

```bash
npm install
npm run dev     # local dev
npm run build   # production build
```

## Source of truth

`src/App.jsx` is canonical. The root-level `hss-slenderness-checker (4).jsx` is a stale artifact from the original Claude-artifact prototype and should not be edited.
