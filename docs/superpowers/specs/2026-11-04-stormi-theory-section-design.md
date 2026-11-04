# Stormi portfolio page — theory & formulas section

## Intent

`projects/stormi-web-application.html` currently describes the app but contains no
mathematics. The full theory lives only in the "Teoria" view of the original app
(`/Users/gbocchi/Documents/GitHub/dev_pers/Boids`). Goal: give the portfolio page a
rigorous "mathematics of the flock" section so a visitor can read the formal model —
vector formulas included — without opening the app.

## Goals

- One new section, **"The mathematics of the flock"**, in English (page language),
  inserted after "How it is built" and before "Tech stack".
- Native **MathML** formulas (zero dependencies, work under `file://`), faithfully
  formalised from `js/boids.js`, covering:
  - neighbour set (perception radius + field-of-view test `cos θ ≥ cos(fov/2)`,
    toroidal minimum-image distance);
  - separation `Σ (p_i − p_j)/d²` over the protection zone (inverse-square);
  - alignment (mean neighbour velocity) and cohesion (centroid) with the shared
    steering operator `steer(u) = limit(normalize(u)·v_max − v_i, f_max)`;
  - weighted sum `a_i = w_s·steer(sep) + w_a·steer(ali) + w_c·steer(coh)`;
  - flee force `w = 2.4·(1 − d/R)` with clamp `4·f_max`, R = 140 px;
  - integration `v ← clamp(v + a, 0.35·v_max, v_max)`, `x ← x + v`;
  - soft-wall force `2.5·f_max·(1 − d/margin)` and toroidal wrap;
  - spatial-grid complexity note O(n) vs O(n²).
- A symbol glossary (v_i, p_i, r_per, r_pro, v_max, f_max, w_s/w_a/w_c, R) so the
  formulas read self-contained.
- Reuse of the three rule diagrams (inline SVG adapted from the app, observatory
  palette) next to their formulas.
- English pseudocode block (translated from the app's theory view).
- A compact parameter guide table for the 9 sliders (from the app's table).
- Short emergence + reading-list note (Reynolds 1987/1999, Cavagna 2010, Shiffman).
- New styles go in `static/css/project-detail.css` (`.project-math`-style classes),
  consistent with the existing "observatory night" design.

## Non-goals

- No KaTeX/MathJax/CDN, no build step, no JS on the page.
- No changes to the Boids app repo or to other portfolio pages.
- Not a verbatim translation of the Italian theory view — adapted and tightened for
  a portfolio reader (the app remains the full teaching view).

## Constraints

- Zero-dependency static site, pages also opened via `file://` (CLAUDE.md).
- Follow existing page patterns: `section.project-section.fade-in`, `project-split`,
  `project-figure`, `project-callout`; shared CSS untouched except additions to
  `project-detail.css`.

## Approaches considered

1. **Native MathML** — zero deps, offline-safe, real math typography in modern
   browsers. **Chosen.**
2. Instrumental monospace/unicode formulas — offline-safe, but weaker for fractions,
   norms and sums; kept as a styling accent for the pseudocode only.
3. KaTeX via CDN — best typography but breaks offline/`file://`; rejected.

## Builder's calls (defaults, correct if desired)

- Section title "The mathematics of the flock"; subsection order: perception →
  three rules → steering → full update step → boundaries → complexity → pseudocode →
  parameters → emergence/refs.
- Formulas centred in "instrument plate" cards; pseudocode in a mono block.
- MathML rendered with a serif math fallback stack; falls back gracefully in old
  browsers (formulas stay readable as linear text).
