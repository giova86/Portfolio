# Stormi theory section Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a dependency-free "The mathematics of the flock" section (native MathML, SVG rule diagrams, pseudocode, parameter table) to `projects/stormi-web-application.html`.

**Architecture:** One new `<section class="project-section fade-in" id="theory">` inserted between the "How it is built" and "Tech stack" sections of the existing page; all new styles appended to `static/css/project-detail.css` as a clearly-commented block using existing `:root` tokens from `shared.css`. No JS, no CDN, no other files touched.

**Tech Stack:** HTML5 + native MathML 4 (browser-rendered), inline SVG, CSS custom properties from `static/css/shared.css`.

**Spec:** `docs/superpowers/specs/2026-11-04-stormi-theory-section-design.md` — the plan argues from the spec; executors read both.

## Global Constraints

- Zero external dependencies: no `cdn.` URLs, no new `<script>`/`<link>` to external hosts (Google Fonts already present is the only external resource; do not add to it). Pages must render correctly opened via `file://`.
- All new copy in English, matching the page's existing tone (see Overview/"How it is built" paragraphs).
- Only two files modified: `projects/stormi-web-application.html` and `static/css/project-detail.css`.
- Reuse existing tokens/classes: `--bg-elev`, `--bg-elev-2`, `--border`, `--border-strong`, `--text`, `--text-muted`, `--text-faint`, `--brand`, `--radius`, `--font-display`, `--font-body`; `section.project-section.fade-in`, `h2`, `h3`, `.project-split`, `.project-figure`, `.project-callout`.
- Formulas must be faithful to `/Users/gbocchi/Documents/GitHub/dev_pers/Boids/js/boids.js` (values quoted below are verbatim from it).

**Verification snippet** (used by Tasks 2–4; run from repo root, expected output `OK`):

```bash
python3 - <<'PY'
from html.parser import HTMLParser
VOID = {'img','br','hr','meta','link','input','source','path','circle','rect','line','use','stop','ellipse','polygon','polyline'}
class P(HTMLParser):
    def __init__(self):
        super().__init__(); self.stack=[]; self.errs=[]
    def handle_starttag(self,t,a):
        if t not in VOID: self.stack.append(t)
    def handle_endtag(self,t):
        if t in VOID: return
        if not self.stack or self.stack[-1]!=t: self.errs.append(f'mismatch </{t}> line {self.getpos()[0]}, open {self.stack[-3:]}')
        else: self.stack.pop()
p=P(); html=open('projects/stormi-web-application.html').read(); p.feed(html)
assert not p.errs, p.errs
assert not p.stack, f'unclosed {p.stack}'
assert 'cdn.' not in html and 'jsdelivr' not in html and 'unpkg' not in html and 'mathjax' not in html.lower(), 'external dependency added'
for need in ['id="theory"','<math','</section>','project-section']:
    assert need in html, f'missing {need}'
print('OK')
PY
```

## Review Focus

- **Old/non-MathML browsers** — formulas must still read as sensible linear text (MathML degrades to its text content); no formula whose meaning exists only in layout.
- **`file://` opening** — section renders fully offline; any added asset must be inline (the SVG diagrams are inline, not `<img src>`).
- **Narrow screens** — wide formulas scroll inside their plate (`overflow-x:auto`) instead of blowing out the page layout.
- **Scroll animations** — the new section uses `.fade-in` so the shared `IntersectionObserver` reveals it; it must not appear permanently invisible (check after scrolling).
- **Neighbouring sections** — stats strip, split sections, gallery and lightbox still work unchanged; the two script tags at the page bottom are untouched.

---

### Task 1: Math styles in `project-detail.css`

**Files:**
- Modify: `static/css/project-detail.css` (append one commented block at the end, before the final `@media (max-width: 768px)` block so media overrides keep working, and add responsive tweaks inside that media block)

**Interfaces:**
- Consumes: tokens/classes listed in Global Constraints.
- Produces (class names Task 2–3 HTML must use exactly): `.math-plate`, `.math-plate > figcaption` (optional label), `.math-note`, `.math-symbols` (a `<dl>`: `dt` symbol, `dd` gloss), `.math-code` (pseudocode `pre>code`), `.project-table` (parameter guide) inside `.table-wrap`, `figure.project-figure svg` styling.

- [ ] **Step 1: Append the styles block** to `static/css/project-detail.css` (place before the last `@media (max-width: 768px)` block):

  - `/* ---- Math / theory section (Stormi) ---- */` comment header.
  - `math { color: var(--text); font-size: 1.06rem; }` and `math { display: block; }` handled via the plate, not globally — keep inline `math` inline.
  - `.math-plate`: `background: var(--bg-elev); border: 1px solid var(--border); border-radius: var(--radius); padding: 1.25rem 1.5rem; margin: 1.25rem 0; overflow-x: auto;` with inner `math { text-align: center; }` centered block layout (`display:block; margin: 0 auto;`).
  - `.math-plate figcaption` (optional label above/below): `font-size: 0.85rem; color: var(--text-faint); text-align: center; margin-top: 0.6rem;`
  - `.math-note`: `font-size: 0.9rem; color: var(--text-muted); font-style: italic;` used under plates for the "why it matters" line.
  - `.math-symbols`: 2-col definition grid — `display:grid; grid-template-columns: auto 1fr; gap: 0.4rem 1.1rem; margin: 1rem 0 1.5rem;`; `dt` in monospace/`--brand` (`font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; color: var(--brand);`), `dd` in `--text-muted`, `font-size: 0.92rem`, `margin:0`.
  - `.math-code`: `pre` treatment — `background: var(--bg-darker); border: 1px solid var(--border); border-radius: var(--radius); padding: 1.25rem 1.5rem; overflow-x: auto;` inner `code`: same monospace stack, `font-size: 0.88rem; line-height: 1.65; color: var(--text); white-space: pre;`
  - `.project-table`: `width:100%; border-collapse: collapse; font-size: 0.92rem;` th/td `text-align:left; padding: 0.6rem 0.8rem; border-bottom: 1px solid var(--border); color: var(--text-muted);` `th { color: var(--brand); font-family: var(--font-display); font-weight:600; white-space: nowrap; }` wrapped in a scrollable `.table-wrap { overflow-x:auto; }`.
  - `figure.project-figure svg`: `width:100%; height:auto; background: var(--bg-elev); border:1px solid var(--border); border-radius: var(--radius-lg); display:block;`
  - Inside the final `@media (max-width: 768px)` block add: `.math-plate { padding: 1rem; }`.

- [ ] **Step 2: Verify CSS integrity**

Run: `python3 -c "css=open('static/css/project-detail.css').read(); assert css.count('{')==css.count('}'), 'unbalanced braces'; [assert c in css for c in ['math-plate','math-symbols','math-code','project-table','table-wrap','math-note']]; print('OK')"`
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add static/css/project-detail.css
git commit -m "feat(stormi): add math section styles to project-detail.css"
```

---

### Task 2: Section skeleton + perception + three rules + steering

**Files:**
- Modify: `projects/stormi-web-application.html` (insert new section after the `How it is built` section closing tag, currently the `</section>` before `<h2>Tech stack</h2>`; and no other changes)

**Interfaces:**
- Consumes: classes from Task 1 (`math-plate`, `math-note`, `math-symbols`, `project-figure` svg, `project-split`).
- Produces: `<section class="project-section fade-in" id="theory">` containing `<h2>The mathematics of the flock</h2>`, `<h3>` subsections in order: `A bird's world` (neighbour set), `The three rules`, `From desire to force: steering`; Task 3 appends further `<h3>` subsections inside this same section.

- [ ] **Step 1: Write the section skeleton + "A bird's world" subsection.** Intro paragraph (2–3 sentences: everything below is formalised from `js/boids.js`, notation with boid *i*, position **p**ᵢ, velocity **v**ᵢ). Then the neighbour set as a `.math-plate` MathML display:

  **N**ᵢ = { j ≠ i : 0 < ‖**Δ**ᵢⱼ‖ ≤ r_per , **Δ̂**ᵢⱼ · **v̂**ᵢ ≥ cos(fov/2) } with **Δ**ᵢⱼ = **p**_j ⊟ **p**ᵢ (toroidal minimum-image difference: per axis, subtract screen size when the raw difference exceeds half the screen) — put the minimum-image detail in a `.math-note` line, plus the FOV explanation ("a neighbour is visible only inside the field-of-view cone; with fov = 360° the test is always true"). Then a `.math-symbols` dl defining: **p**ᵢ, **v**ᵢ, r_per (perception radius), r_pro (protection radius), fov, v_max (max speed), f_max (maxForce / Reattività), w_s w_a w_c (rule weights), R (predator threat radius, 140 px), m (wall margin, 90 px).

- [ ] **Step 2: Write "The three rules" subsection** — one intro sentence, then three rows, each a `div.project-split` (text div + `figure.project-figure` with the inline SVG copied from `/Users/gbocchi/Documents/GitHub/dev_pers/Boids/index.html`: separation `index.html:183–194`, alignment `205–217`, cohesion `227–240`, keep `viewBox`, strokes/fills and `figcaption`s — translate captions to English), with the formula as a `.math-plate` under the text:
  - **Separation:** **s**ᵢ = Σ_{j∈Pᵢ} (**p**ᵢ ⊟ **p**_j)/‖**Δ**ᵢⱼ‖², Pᵢ = { j : ‖**Δ**ᵢⱼ‖ < min(r_pro, r_per) } — note: inverse-square, "the push grows as 1/d² when a neighbour closes in".
  - **Alignment:** **ū** = (1/|Nᵢ|) Σ_{j∈Nᵢ} **v**_j — note: mean neighbour velocity.
  - **Cohesion:** **c**ᵢ = **p**ᵢ + (1/|Nᵢ|) Σ_{j∈Nᵢ} **Δ**ᵢⱼ (centroid), desire **c**ᵢ − **p**ᵢ — note: centre of mass of neighbours.

- [ ] **Step 3: Write "From desire to force: steering" subsection.** The shared steering operator as the centrepiece plate:

  steer(**u**) = limit( (**u**/‖**u**‖)·v_max − **v**ᵢ , f_max ),  limit(**x**, c) = **x** if ‖**x**‖ ≤ c else (**x**/‖**x**‖)·c

  Reference MathML pattern for one plate (implement the rest in this style; use `<mo>`, `<mi>`, `<msub>`, `<mfrac>`, `<munderover>` for Σ, `<mrow>`, `<mspace>`; mark bold vectors with `<mi mathvariant="bold">`):

  ```html
  <div class="math-plate">
    <math display="block" xmlns="http://www.w3.org/1998/Math/MathML">
      <mrow><mtext>steer</mtext><mo>(</mo><mi mathvariant="bold">u</mi><mo>)</mo>
        <mo>=</mo><mtext>limit</mtext><mo>(</mo>
        <mfrac><mi mathvariant="bold">u</mi><mrow><mo>‖</mo><mi mathvariant="bold">u</mi><mo>‖</mo></mrow></mfrac>
        <mo>·</mo><msub><mi>v</mi><mtext>max</mtext></msub><mo>−</mo>
        <msub><mi mathvariant="bold">v</mi><mi>i</mi></msub><mo>,</mo>
        <msub><mi>f</mi><mtext>max</mtext></msub><mo>)</mo></mrow>
    </math>
    <figcaption>desire minus state, capped at the agility budget</figcaption>
  </div>
  ```

  Paragraph after the plate: `f_max` is the "Reattività" slider — low values give wide majestic turns, high values snappy reactions (adapt from the app's steering section, `/Users/gbocchi/Documents/GitHub/dev_pers/Boids/index.html:245–250`).

- [ ] **Step 4: Run the verification snippet** (from Global Constraints) plus check no external URLs were added:
  Run: `grep -c "http" projects/stormi-web-application.html` and compare to before the change (only pre-existing fonts/OG entries allowed; count of *new* lines containing `http` must be 0).
  Expected: `OK`, no new external URLs.

- [ ] **Step 5: Commit**

```bash
git add projects/stormi-web-application.html
git commit -m "feat(stormi): add math section with three rules and steering"
```

---

### Task 3: Full update, boundaries, complexity, pseudocode, parameters, emergence

**Files:**
- Modify: `projects/stormi-web-application.html` (append `<h3>` subsections inside the existing `#theory` section, after Task 2's steering subsection)

**Interfaces:**
- Consumes: Task 2's `#theory` section, Task 1's `math-code`, `project-table`, `table-wrap`, `project-callout`.
- Produces: completed `#theory` section; nothing downstream consumes it.

- [ ] **Step 1: Add "One tick, from top to bottom" subsection** — the weighted sum plate **a**ᵢ = w_s·steer(**s**ᵢ) + w_a·steer(**ū**) + w_c·steer(**c**ᵢ − **p**ᵢ), then the integration plate **v**ᵢ ← clamp-speed(**v**ᵢ + **a**ᵢ·Δt) with 0.35·v_max ≤ ‖**v**ᵢ‖ ≤ v_max (note: a bird never freezes, never outruns the frame; Δt is the fixed tick, scaled by ½×/¼× slow motion), then the flee plate: for predator **P**, if dᵢ = ‖**p**ᵢ − **P**‖ < R then **a**ᵢ += 2.4·(1 − dᵢ/R)·limit( ((**p**ᵢ−**P**)/dᵢ)·v_max − **v**ᵢ , 4·f_max ) — note: weight fades to 0 at the rim of the 140 px threat ring, force budget is 4× a rule.

- [ ] **Step 2: Add "Boundaries" subsection** (short): toroidal wrap plate **p**ᵢ ← (**p**ᵢ mod W, **p**ᵢ mod H) — "the screen is a torus, and the neighbour maths follows the wrap"; soft walls plate: inside margin m of an edge, Δv_∥ = 2.5·f_max·Δt·(1 − d_∥/m) toward the interior — "an invisible valley wall".

- [ ] **Step 3: Add "Why hundreds of birds stay cheap" note block** — all-pairs search is O(n²); the flock is bucketed into a grid of cells sized max(28 px, r_per) and each bird inspects only the 3×3 cell block around it (wrap-aware), O(n) in practice. Use a `.math-note` or small `.project-callout`.

- [ ] **Step 4: Add "The loop in pseudocode" subsection** — `pre.math-code > code`, English translation of the app's pseudocode (`/Users/gbocchi/Documents/GitHub/dev_pers/Boids/index.html:255–267`), keeping its exact structure (neighbours → sep/ali/coh → weighted steer → flee → limit → integrate → wrap/walls). Escape `<` as `&lt;` inside `pre`.

- [ ] **Step 5: Add "The nine knobs" subsection** — `div.table-wrap > table.project-table`, 4 columns (Parameter / What it sets / Raise it / Lower it), 9 rows translated from the app table (`/Users/gbocchi/Documents/GitHub/dev_pers/Boids/index.html:275–291`): Individuals, Max speed, Reactivity (maxForce), Cohesion, Alignment, Separation, Perception radius, Protection zone, Field of view.

- [ ] **Step 6: Add "Emergence, and where to read more" subsection** — short emergence paragraph (no bird knows the flock's shape; scale-free correlations, Cavagna et al. 2010, ~7 topological neighbours) and a `ul.project-features` reading list: Reynolds 1987 SIGGRAPH; Reynolds 1999 Steering Behaviors (GDC); Cavagna et al. 2010 PNAS; Shiffman, The Nature of Code ch. 5.

- [ ] **Step 7: Run the verification snippet**
  Expected: `OK`

- [ ] **Step 8: Commit**

```bash
git add projects/stormi-web-application.html
git commit -m "feat(stormi): complete theory section with update step, pseudocode, parameter guide"
```

---

### Task 4: Visual verification

**Files:** none (verification only; fixes go into the two Task 1–3 files)

**Interfaces:** consumes the finished page.

- [ ] **Step 1: Serve and view desktop.** `python3 -m http.server 8014 &` then open `http://localhost:8014/projects/stormi-web-application.html`. Check: theory section renders between "How it is built" and "Tech stack"; every MathML plate shows real math (fractions stacked, Σ with bounds), the three SVG diagrams show the observatory palette, pseudocode scrolls horizontally on overflow, parameter table renders, fade-in reveals the section on scroll.
- [ ] **Step 2: Mobile width + offline.** Same URL at ~390 px viewport: no horizontal page scroll (plates scroll internally). Then open `file:///Users/gbocchi/Documents/GitHub/dev_pers/Portfolio/projects/stormi-web-application.html`: section renders fully offline (fonts may fall back, math must not).
- [ ] **Step 3: Regression.** Console clean (no errors); gallery still populates and lightbox opens; all pre-existing sections unchanged. Kill the http server.
- [ ] **Step 4: If anything failed,** fix in the two allowed files, re-run Task 1 Step 2 check + verification snippet, re-verify, then:

```bash
git add -A projects/stormi-web-application.html static/css/project-detail.css
git commit -m "fix(stormi): theory section rendering polish"
```

(Commit only if fixes were needed.)
