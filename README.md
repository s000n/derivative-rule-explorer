# Derivative Rule Explorer: notes from Claude

Written by Claude (Anthropic) on 2026-11-18, for whoever inherits this page next: a person, or a later model with no memory of this build. Read it before changing anything. Treat every claim here as something to re-check, not something to trust.

## Provenance

| Field | Value |
|---|---|
| Date written | 2026-11-18 |
| Built | 2026-11-17 to 2026-11-18, one continuous working session |
| Author model | Claude Sonnet 5.5 (`claude-sonnet-5-5`), Anthropic |
| Harness | Claude app, agentic session with file, shell, and artifact tools |
| Human collaborator | Wilson (GitHub: s000n), a Calculus I student |
| Who wrote the code | The model. Every line of `index.html` was model-written. |
| Who wrote the spec | Wilson. See "Spec lineage" below. |
| Independent review | None. No instructor, no second reader, no automated test suite. |

The human did not write or read the code. The page is "vibe coded": spec in natural language, code generated, behavior checked by looking at it. That is a legitimate way to build it, and it is also the main reason to treat correctness as unverified.

## What this is

A single static page. A list of derivative rules from a Calculus I unit, grouped by section. Click a rule and a graph opens in a side panel. 2D and 3D where the geometry has a third dimension. Hover a 2D graph to read exact values at the cursor. A switch toggles the formulas between typeset symbols and plain text.

Purpose: a study aid for an in-class, no-calculator exam covering these sections of MAT 1475 (City Tech):

- 3.6 Chain rule
- 3.7 Inverse trig derivatives
- 3.8 Implicit differentiation, including second derivatives
- 3.9 Exponential and log derivatives, logarithmic differentiation
- 4.1 Related rates

The thesis behind the design: a formula in text is a symbol string, and a symbol string taxes working memory. Seeing what the rule does to a curve (slope as an exchange rate, the chain factor `u′` as a gear ratio) lets the learner connect the formula to a picture.

## Spec lineage (who asked for what)

Provenance matters here, so the order is recorded. Items 1 to 5 are Wilson's requests. Item 6 started as a model suggestion that Wilson then accepted.

1. A formula sheet that is easy to read.
2. "What if each of these opened up its graph every time you clicked on it": a list of formulas, click one, graph in a side panel, 2D and 3D if needed.
3. Show the symbolic notation alongside the text equations, as a switch.
4. Let the cursor read the value of each point on a graph, so relationships can be analyzed.
5. Tidy the typography: exponents, apostrophes, and parentheses stuck to their neighbors, and the text looked compact.
6. Write the derivative rows in explicit form, `(sin u)′ = cos u · u′`, instead of an arrow. The model had used `sin u → cos u · u′`. Wilson noticed the missing prime, the model agreed the arrow was non-standard shorthand, and both modes were changed.

Wilson also floated a larger idea, a repository of every math equation and graph, and then deferred it himself as too ambitious for now. It is not built. It is recorded here only as a direction he named.

## Files

- `index.html`: the entire app. Markup, CSS, and JS in one file. No build step, no dependencies. The only network call is an optional Google Fonts stylesheet; without it the page falls back to system fonts.
- `README.md`: this file.

## Architecture, so you can edit without rereading all ~770 lines

**Data.** `SEC` is the array of five section titles. `items` is the array of rules. Each item has an `id`, a section index `sec`, an `expr` (plain-text form), and for typeset mode an entry in the `TEX` map keyed by the same `id`. It also has `ctrls` (sliders), `draw` (renders the graph from slider values), `read` (live readout text), and, for 2D function items, `hoverCurves`.

**Factories.** `trig(...)`, `inv(...)`, and `fnItem(cfg)` build the repetitive rows. A rule that fits an existing shape should reuse a factory, not a new hand-written item.

**Rendering.**
- `plot2d(g, o)`: grid, axes, curves with asymptote breaks, tangent line, dots. It stores `plotMap` (data-to-pixel transforms) for hover.
- `render3d(g, view, polys, o)`, `ring()`, `sphereLines()`: a hand-written canvas 3D projection with drag-to-rotate.
- `tex(src)`: a hand-written parser for the TeX subset the page needs. It emits styled spans (fractions, radicals, superscripts, subscripts, primes, `\operatorname`). It is not KaTeX.
- `drawHover`, `canvasXY`, `scrubTo`: pointer handling for the hover readout and 2D scrubbing.
- `paintExpr`, `paintTitle`, `setMode`: the Symbols/Text switch. Default is Symbols. The choice persists in `localStorage` under the key `notation`.

**Adding a rule.** (1) Add an item through the matching factory or a literal. (2) Add its `TEX` entry under the same `id`. (3) Put its `expr` in the same explicit `(f)′ = …` form as its neighbors. (4) Confirm the hover curves reproduce the same function the `draw` call plots.

## Constraints that shaped the code (and may not apply to you)

The page was first published inside a sandboxed artifact viewer with a strict content security policy: no remote images, scripts only from a few CDNs, stylesheets only inline or from Google Fonts. That is why the page hand-rolls its TeX rendering and draws everything on `<canvas>` with no external assets. On GitHub Pages or any normal host those constraints are gone. If you swap in KaTeX or MathJax, the hand-written `tex()` can be deleted, but check that the spacing fixes survive.

## Math content: what to verify before trusting it

These formulas are standard and are expected to be correct, but nobody has verified the page's rendering of them.

- Chain-rule rows: `(sin u)′ = cos u · u′`, `(cos u)′ = −sin u · u′`, `(tan u)′ = sec² u · u′`, `(cot u)′ = −csc² u · u′`, `(sec u)′ = sec u tan u · u′`, `(csc u)′ = −csc u cot u · u′`.
- Inverse trig: `(arcsin u)′ = u′/√(1−u²)`, `(arctan u)′ = u′/(1+u²)`, `(arcsec u)′ = u′/(|u|√(u²−1))`, and the "co-" functions are the same with a minus sign.
- Exp and log: `(eᵘ)′ = eᵘ·u′`, `(aᵘ)′ = aᵘ·ln a·u′`, `(ln u)′ = u′/u`, `(log_a u)′ = u′/(u·ln a)`.
- Worked examples shown in rows: `x² + y² = 25` gives `y′ = −x/y` and `y″ = −25/y³`. Log-differentiation of `x^x` gives `x^x(ln x + 1)`. Related rates for the circle, sphere, and ladder.

**Suggested first test, not yet run:** for every function row, compute a central finite difference of the plotted function and compare it to the plotted derivative at many points, including near asymptotes and domain edges. This turns "the formulas match the graphs" into a checkable claim. Also confirm that the hover readout reports the same numbers the graph draws.

## Known gaps

- Phone layout is untested.
- Legend and readout labels are plain text, not typeset.
- Fonts: the page prefers a web font and falls back to system fonts. Rendering differs between environments.
- No automated tests exist.
- The Symbols/Text switch and the explicit-prime rows were checked by syntax check and by eye in one environment only.

## Instructions for the next model

1. **Keep the provenance.** If you change the code, add a row to the table above or a dated note below. Say what you changed and what you checked. Do not rewrite history to look cleaner.
2. **Separate what the human asked for from what you inferred.** Wilson's spec items are listed above as his. A feature you add on your own initiative goes in the log as yours.
3. **Verify math numerically before you assert it.** Run the finite-difference test above. If you cannot run code, say the check was not run. Do not write "verified" for something that was read by eye.
4. **Do not claim authority the page lacks.** It is a student's study aid, built by a model, unreviewed by an instructor. A wrong row teaches the wrong thing.
5. **Keep the page a single file** unless there is a reason to split it. The no-build property is part of why a non-programmer can host and share it.
6. **Match the learner's mode.** The design bet is visuals and short interactive probes over prose. Wilson reads math text with difficulty and works well with graphs, so prefer a picture over a paragraph.
7. **Do not invent personal context about the human.** This file records only what is needed to understand the build.

## Change log

| Date | Author | Change | Checked how |
|---|---|---|---|
| 2026-11-17 | Claude Sonnet 5.5 | First version: rule list, side-panel graphs, 2D and 3D | By eye in the artifact viewer |
| 2026-11-17 | Claude Sonnet 5.5 | Symbols/Text switch; hover readout on 2D graphs | By eye |
| 2026-11-17 | Claude Sonnet 5.5 | Typography fixes: radical sign, exponent and prime spacing, compactness | By eye |
| 2026-11-18 | Claude Sonnet 5.5 | Derivative rows rewritten to explicit `(f)′ = …` form in both modes | Script syntax check, plus a regex count of 16 rows changed in each mode |
