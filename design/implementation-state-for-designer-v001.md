# LeadPilot interface — implementation state, for the designer

> **RESOLVED 2026-07-14:** the designer's artifact arrived
> (`mockups/leadpilot_glass_reference.html`) and its values were
> ported 1:1 the same day — every suspect below was confirmed real
> (inset top-sheen, Inter-first font stack, richer background stack
> with luminance ramp + neutral orb + vignette, 1560px layout
> ceiling), plus component details the prose spec never captured
> (segmented tab control, accent-light card headers, neutral R2/R3
> pills, amber urgent-approve button, avatar in the lead header).
> `assets/*.png` now shows the post-port state. Two deliberate
> deviations from the artifact, both Marc's later decisions:
> accent-tinted hover states (A14 — artifact hovers are white-alpha)
> and drag-resizable panes (artifact has fixed widths; its 240/280px
> became our defaults). This doc is kept for the history/method; the
> artifact is the styling source of truth going forward.

**Date:** 2026-07-14
**Audience:** the designer (Marc's claude.ai design session) whose
mockups produced `leadpilot_interface_design_spec_v001.md`.
**Problem this doc exists to solve:** the built interface follows the
spec's written values but still doesn't look like the mockups. Marc
confirmed the gap visually. We need the *actual mockup artifacts* to
close it — see "The ask" at the bottom.

Current-state screenshots (1680×1000, headless Chromium, cool-blue
theme, default orbs background): `assets/login.png`,
`assets/workspace.png` (nothing selected), `assets/lead.png` (lead
open with an urgent handoff card).

## What this implementation is

Not a mockup — the real Step 3 product: server-rendered Jinja2 +
htmx partial swaps on FastAPI (stack locked by Decision 022, no SPA,
no build pipeline). Every approve/reject button drives the real
approval-gate state machine. All styling lives in one hand-written
stylesheet: `leadpilot/src/leadpilot/static/css/app.css`. System font
stack (no webfonts currently — worth checking against the mockup, see
suspects below).

## Exactly what's implemented today (current values, verbatim)

**Base layer** — page `#0c0d10` untinted; fixed background layer:

```css
radial-gradient(circle 30vw at 15% 20%, rgba(var(--ar), 0.20), transparent 70%),
radial-gradient(circle 34vw at 88% 80%, rgba(var(--ar), 0.14), transparent 70%)
```

(Marc's relayed spec said `circle 280px`/`circle 320px`; at desktop
viewport widths those were near-invisible dots, so with Marc's
approval the radii became viewport-relative — 280/320px ≈ 30/34vw of
a ~930px mockup artboard. Positions and alphas are untouched.)

Four alternate rep-selectable patterns (grid / weave / corner / none)
exist as personalization on the same system; orbs is the default.

**Surfaces** (never accent-tinted):
- Panels: `rgba(255,255,255,0.035)`, `backdrop-filter: blur(18px)`,
  `1px solid rgba(255,255,255,0.08)`, radius 12px
- Nested cards: `rgba(255,255,255,0.03)`, `1px solid rgba(255,255,255,0.07)`, radius 12px
- No panel is borderless. **No panel currently has any box-shadow or
  inner highlight** — see suspects.

**Accent:** one variable `--ar` (RGB triple) per theme + a light-accent
text color. Seven themes as specced (§2 table, exact RGB values).
Primary buttons: `rgba(var(--ar), 0.18)` bg, `1px solid rgba(var(--ar), 0.5)`,
light-accent text — no solid fills. Semantic green/amber/red fixed
across themes.

**Text ladder:** `#fff` headings/names · `#e9e7e2` body · `#9a9ea6`
secondary · `#5b6067` micro-labels (11px, uppercase, 0.1em tracking).

**Click feedback:** `:active` scale(0.96); 180ms glow ring
`0 0 0 1px rgba(var(--ar),0.7), 0 0 12px rgba(var(--ar),0.3)`.

**Layout:** three-pane resizable workspace (≈200px queue / flex
center capped at 900px / ≈250px context rail), 6px drag handles,
panels hug their content height rather than stretching to the
viewport (they originally stretched full-height, which buried the
background entirely — that was fixed after Marc's first "muddled"
report).

## What was tried, in order (so nothing gets re-litigated)

1. **v1:** low-alpha multi-pattern backgrounds (0.07–0.25), panels
   stretched full viewport height. Result: panels covered ~97% of the
   background; everything blurred to uniform near-black. Marc:
   "backgrounds all look the same, not polished."
2. **v2:** stronger multi-stop gradients + neutral base luminance
   lift + film grain. Marc then supplied the written glass-system
   values ("do not improvise") — v2 removed wholesale.
3. **v3 (current):** Marc's exact system applied. Two structural
   fixes on top, both visually confirmed via screenshots: panels now
   hug content so the base layer is actually exposed, and orb radii
   went viewport-relative. Marc: closer, but "still looks nothing
   like what the designer showed me."

## Likely suspects for the remaining gap

The written spec appears to be a lossy transcription of the mockup.
Things a claude.ai HTML mockup almost certainly had that the spec
never mentioned, and which the implementation therefore lacks:

1. **Depth/shadow on glass.** Our panels have zero `box-shadow` — no
   drop shadow lifting them off the background, no inset top
   highlight (`inset 0 1px 0 rgba(255,255,255,0.06)`-style sheen).
   Flat hairline-only glass reads muddy. This is my top suspect.
2. **Typeface.** We're on the system font stack; the mockup was
   probably Inter (or similar) with tighter heading tracking —
   changes perceived polish a lot.
3. **Background richness.** Two orbs at 0.20/0.14 may be only part of
   the mockup's stack — mockups often add a subtle vertical
   luminance ramp or vignette under the orbs (an earlier build here
   had one; it was removed as unspecced).
4. **Artboard proportions & density.** A mockup demos with dense,
   hand-placed content in a fixed frame; the real app renders
   whatever data exists at real viewport sizes, so emptier states
   look starker.
5. **Chrome details** the spec didn't cover: header treatment,
   scrollbar styling, chip/badge geometry, exact icon usage.

## The ask

Prose descriptions keep losing information. Please export from the
design session and drop into `leadpilot-docs/design/`:

1. **The actual mockup artifact(s)** — the full HTML/CSS of the
   three-pane workspace and theme-lab mockups Marc approved, not a
   description of them. The implementation can then port values 1:1
   (it's one stylesheet; swapping values is cheap and safe — the
   behavior layer is untouched by styling changes).
2. If exporting isn't possible: the mockup's **complete CSS for one
   glass panel** (every property: background, border, shadows,
   pseudo-elements), the **complete body/background stack**, and the
   **font-family + weights** used.
3. A screenshot of the mockup at a stated viewport size, so there's
   an unambiguous target to compare `assets/*.png` against.

Constraints any revision must live within (from the locked stack and
the spec's own rules): server-rendered htmx (no SPA), the four
sanctioned JS exceptions, hours-of-reading legibility, semantic
colors never re-themed, and the approval-gate interaction anatomy is
not up for restyle-driven change.
