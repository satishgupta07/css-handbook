# CSS — Concepts & Theory Guide

A theory-first walkthrough of CSS — from the cascade to container queries.
The lessons are short, hands-on demos; this README is the conceptual
scaffolding behind them. Read a section, then open the lesson(s) to see
the ideas live.

---

## 1. What CSS really is

**CSS** stands for **Cascading Style Sheets**. It's a **declarative** language: you describe *what* the page should look like, and the browser figures out *how* to paint it.

> *"CSS is a language for telling the browser **what** you want, not **how** to make it."*

The basic shape of a rule:

```css
selector {
  property: value;     /* one declaration */
  property: value;     /* another declaration */
}
```

- The **selector** picks which elements the rule applies to.
- Each `property: value` pair is a **declaration**.
- A selector + its declarations is a **rule**.
- Comments only use `/* ... */` — `//` is **not** valid CSS.

### How styling actually reaches an element

CSS doesn't "apply" a single rule. The browser builds, for every element, a **computed style** — the final value of every property — by running through three pipelines:

```
1. CASCADE     →  many rules match, who wins per property?
2. INHERITANCE →  some properties flow from parent if not set
3. DEFAULTS    →  user-agent stylesheet fills in the rest
```

Understanding the cascade is the single biggest conceptual leap in CSS. Phases 1–2 of this guide are entirely about it.

---

## 2. Three ways to attach CSS (`01_foundations/`)

| Method | Looks like | Use when |
|--|--|--|
| External | `<link rel="stylesheet" href="page.css">` | Almost always — cacheable, reusable, scales. |
| Internal | `<style>...</style>` in `<head>` | One-page demos, critical inline CSS. |
| Inline | `<p style="color:red">` | Avoid in production — high specificity, untestable, mixes concerns. |

Specificity-wise, inline beats internal/external — but that's a side effect, not a reason to use it.

---

## 3. The cascade — how rules are picked

When two rules touch the same property on the same element, the cascade decides who wins. The algorithm runs in this order:

```
1. ORIGIN & IMPORTANCE   (author normal < author !important, etc.)
2. SPECIFICITY           (higher score wins)
3. SOURCE ORDER          (later in the file wins)
```

### Origin & importance

Three sources of CSS exist:
- **User-agent** — the browser's default stylesheet (`<h1>` is big, links are blue).
- **User** — site styles the user installs (rare).
- **Author** — your stylesheet.

Normally **author > user > user-agent**. With `!important`, the order partly *reverses* — but for everyday code you can think of it as: `author !important` beats `author normal`.

`!important` is a code smell. It locks future contributors out and triggers an arms race. Reach for it only for real overrides (utility classes, third-party widget escapes).

### Specificity — the 4-tuple

Score every selector as `(a, b, c, d)`:

| Slot | What counts | Example |
|--|--|--|
| `a` | Inline `style=""` | `1,0,0,0` |
| `b` | id selectors | `#header` → `0,1,0,0` |
| `c` | class, attribute, pseudo-class | `.btn`, `[type=text]`, `:hover` → `0,0,1,0` |
| `d` | element, pseudo-element | `p`, `::before` → `0,0,0,1` |

Compare left-to-right. `0,1,0,0` beats `0,0,99,0`. The universal selector `*` and combinators (`>`, `+`, `~`) score 0.

### Source order

When everything else ties, **the rule that comes later wins**. This is also why "the last imported stylesheet wins" — it's just source order at a different scale.

### Inheritance

Some properties flow from parent to child by default — the **inherited** ones — and some don't.

| Inherit by default | Don't inherit |
|--|--|
| `color`, `font-family`, `font-size`, `line-height`, `text-align`, `visibility`, `cursor` | `width`, `height`, `padding`, `margin`, `border`, `background`, `display`, `position` |

Override with the **magic keywords**:
- `inherit` — copy from parent.
- `initial` — reset to the spec's default.
- `unset` — `inherit` if the property naturally inherits, otherwise `initial`.
- `revert` — fall back to the user-agent stylesheet.

---

## 4. Selectors — picking elements (`02_selectors/`)

Mastery of selectors = mastery of CSS. The lessons split them into four families.

### Basic & attribute selectors

| Selector | Matches |
|--|--|
| `*` | Everything |
| `h1` | All `<h1>` elements |
| `.badge` | All elements with class `badge` |
| `#lead` | The element with id `lead` |
| `h1, h2, h3` | Group — match any in the list |
| `[disabled]` | Has the attribute (any value) |
| `[type="text"]` | Exact match |
| `[class~="card"]` | Word match (one of the space-separated values) |
| `[href^="https"]` | **Starts with** |
| `[href$=".pdf"]` | **Ends with** |
| `[href*="example"]` | **Contains** |
| `[lang|="en"]` | `en` or `en-*` (BCP 47 prefix) |
| `[type="text" i]` | Case-**i**nsensitive match |

Attribute selectors shine for styling form inputs by type and for reading `data-*` hooks: `[data-state="open"]`.

### Combinators — relationships in the DOM

| Combinator | Meaning |
|--|--|
| `A B`  (space) | Descendant — any `B` inside `A` (any depth) |
| `A > B` | Child — `B` that is a *direct* child of `A` |
| `A + B` | Adjacent sibling — `B` immediately after `A` |
| `A ~ B` | General sibling — any `B` after `A`, same parent |

### Pseudo-classes — element *state* (single colon `:`)

State-based:
- `:hover`, `:focus`, **`:focus-visible`** (keyboard-only focus — the modern way to keep accessible focus rings without showing them on every mouse click), `:active`, `:visited`.
- `:disabled`, `:checked`, `:placeholder-shown`, `:required`, `:optional`, `:valid`, `:invalid`.

Structural — based on position among siblings:
- `:first-child`, `:last-child`, `:only-child`, `:empty`.
- `:nth-child(n)`, `:nth-of-type(n)`. The argument is a formula: `2n` (even), `2n+1` (odd), `3` (third), `n+4` (fourth onward).

Functional — take a selector list:
- `:not(.foo)` — anything that doesn't match.
- `:is(h1, h2, h3)` — shortcut for grouping; specificity = the *highest* inside.
- `:where(...)` — same as `:is()` but **specificity 0** (perfect for resets).
- `:has(.unread)` — the long-awaited **parent selector** (Phase 11).

### Pseudo-elements — virtual children (double colon `::`)

| Pseudo-element | Use |
|--|--|
| `::before`, `::after` | Inject content (must declare `content: "..."`). Source of CSS-only icons, badges, tooltips, decorative shapes. |
| `::first-letter` | Drop caps. |
| `::first-line` | First line of a paragraph (responsive — adapts on resize). |
| `::placeholder` | Placeholder text style. |
| `::selection` | Highlighted text. |
| `::marker` | List bullets / numbers. |

You can pull data into generated content: `content: attr(data-label)`.

---

## 5. The Box Model (`03_box_model/`)

Every element is a rectangular box of four concentric layers:

```
┌── margin ─────────────────────────────────┐
│   ┌── border ─────────────────────────┐   │
│   │   ┌── padding ─────────────────┐   │   │
│   │   │                            │   │   │
│   │   │         content            │   │   │
│   │   │                            │   │   │
│   │   └────────────────────────────┘   │   │
│   └────────────────────────────────────┘   │
└────────────────────────────────────────────┘
```

- **content** — the text/image itself.
- **padding** — clear space *inside* the border (background color extends here).
- **border** — visible edge.
- **margin** — clear space *outside* the border (transparent — the parent's background shows through).

### `box-sizing` — the most important reset

By default, `box-sizing: content-box` means `width: 200px` applies to the **content only** — padding and border get *added on top*. So a "200px" box with `padding: 20px` and `border: 2px` is actually **244px** wide. This is the source of countless layout bugs.

The universal modern reset:

```css
*, *::before, *::after { box-sizing: border-box; }
```

Now `width: 200px` means **the box is 200px**. Everything fits.

### Shorthand — clockwise from top

```css
margin: 10px;                   /* all sides */
margin: 10px 20px;              /* vertical | horizontal */
margin: 10px 20px 30px;         /* top | horizontal | bottom */
margin: 10px 20px 30px 40px;    /* top | right | bottom | left */
```

`padding` shorthand follows the same rules.

### Margin collapsing

**Vertical margins between adjacent block siblings collapse** — the gap is the *larger* of the two, **not the sum**. Surprising, but intentional (it produces consistent paragraph spacing).

```
<p style="margin-bottom: 30px">A</p>
<p style="margin-top: 20px">B</p>
↓
gap = 30px (not 50px)
```

It also collapses between a parent and its first/last child — *unless* something separates them: a border, padding, `overflow: hidden`/`auto`, `display: flex`/`grid`, or `display: flow-root`.

It does **not** happen on flex/grid items, floats, or positioned elements — another reason flex/grid feel "saner" than the old block flow.

**Negative margins** are legal and pull elements toward each other — useful for "bleed" overlap effects.

---

## 6. Visual styling (`04_visual/`)

### Colors

| Format | Looks like | When |
|--|--|--|
| Named | `red`, `tomato` | Quick demos. |
| Hex | `#ff5733`, `#f53` (short), `#ff573380` (with alpha) | Most common. |
| `rgb()` | `rgb(255 87 51)`, `rgb(0 0 0 / 0.5)` | Modern syntax — no commas needed. |
| `hsl()` | `hsl(12 100% 60%)` | Best for *tweaking* (nudge hue/sat/light independently). |
| `oklch()` | `oklch(0.6 0.2 30)` | Perceptually uniform — equal numeric jumps look like equal visual jumps. The 2024+ standard. |

**Magic values**:
- `transparent` — fully see-through.
- `currentColor` — the element's own `color`. Makes borders, SVG fills, and shadows track text color automatically. Hugely useful.
- `inherit` — copy parent's color.

### Backgrounds & gradients

The `background` shorthand wraps eight sub-properties:

```css
background:
  url("hero.jpg") center / cover no-repeat fixed;
/*  image          position size  repeat   attachment */
```

Sub-properties:
- `background-color`, `background-image` (URL or gradient).
- `background-size` — `cover` (fill, may crop), `contain` (fit, may letterbox), `100px`, `100% auto`.
- `background-position` — `center`, `top right`, `50% 30%`.
- `background-repeat` — `no-repeat`, `repeat-x`, `space`, `round`.
- `background-attachment` — `scroll`, `fixed` (parallax), `local`.
- `background-clip` — `border-box`, `padding-box`, `content-box`, `text` (text-clipped backgrounds — gradient text).
- `background-origin` — where positioning measures from.

**Layered backgrounds** — comma-separated; **first listed appears on top**. The classic dark-image-overlay:

```css
background:
  linear-gradient(rgba(0,0,0,.5), rgba(0,0,0,.5)),
  url("photo.jpg") center/cover no-repeat;
```

**Gradients** are images — they go in `background-image`:
- `linear-gradient(45deg, red, blue)` — angle, then color stops.
- `radial-gradient(circle at top right, ...)`.
- `conic-gradient(from 0deg, red, blue, red)` — pie charts, color wheels.
- `repeating-*-gradient(...)` — stripes and patterns.

### Borders, radius, outline

`border: width style color` (e.g. `1px solid #ccc`). Per-side variants: `border-top`, etc.

Styles: `solid`, `dotted`, `dashed`, `double`, `groove`, `ridge`, `inset`, `outset`.

`border-radius` rounds corners:
- `border-radius: 8px` — all four corners.
- `border-radius: 999px` — pill (when wider than tall).
- `border-radius: 50%` — circle (on a square).
- Per-corner: top-left, top-right, bottom-right, bottom-left (clockwise from TL).
- `border-radius: 30px / 60px` — elliptical (horizontal / vertical radius).

**`outline`** is *not* a border — it sits **outside** the box, doesn't take space in layout, and doesn't have per-side variants. Critical for **accessibility**: it's the focus ring. Use `outline-offset` to push it out.

> Never `outline: none` without an alternative. You're disabling keyboard focus for sighted users.

### Shadows

```css
box-shadow: 0   1px  3px      rgba(0,0,0,.1)        ;
/*          x   y    blur     color                */
box-shadow: 0   0    0  3px   #6366f1   inset       ;
/*                          spread     color   inset */
```

- `inset` makes the shadow paint **inside** the element.
- `spread` (the 4th length) grows the shadow without blur — perfect for outlined rings.

**Modern UIs stack multiple soft shadows** for natural depth, instead of one hard shadow:

```css
box-shadow:
  0 1px 3px rgba(0,0,0,.05),
  0 8px 24px rgba(0,0,0,.10);
```

`text-shadow` is the same recipe (no spread, no inset) for text.

Shadows are expensive to **animate** — prefer `transform` and `opacity`.

---

## 7. Typography (`05_typography/`)

### Font stacks & web fonts

Always provide a **fallback chain** ending in a generic family — so the page works if the custom font fails to load:

```css
font-family: 'Inter', system-ui, -apple-system, Segoe UI, sans-serif;
```

Generic families: `serif`, `sans-serif`, `monospace`, `cursive`, `system-ui` (the OS's default UI font — fast, no download).

Web fonts come in two flavors:
- `<link href="https://fonts.googleapis.com/css2?family=Inter&display=swap">` — easy, third-party.
- Self-hosted via `@font-face` — more control, no third-party request.

**`font-display: swap`** (or `&display=swap` in Google URLs) shows the fallback immediately and swaps when the custom font loads — avoids invisible text (the dreaded FOIT).

The `font` shorthand sets several properties at once:

```css
font: italic bold 1rem / 1.5 'Inter', sans-serif;
/*    style  weight size  line-height family       */
```

### Text properties

| Property | What it controls |
|--|--|
| `color` | Foreground (text) color |
| `text-align` | `left`, `right`, `center`, `justify`, `start`, `end` |
| `text-decoration` | `underline`, `line-through`, plus `-style`, `-color`, `-thickness` |
| `text-transform` | `uppercase`, `lowercase`, `capitalize` |
| `letter-spacing` | Tracking — space between letters |
| `word-spacing` | Space between words |
| `line-height` | Leading — vertical rhythm. **Unit-less is best** (`1.5` scales with font-size). |
| `text-indent` | First-line indent |
| `white-space` | `normal`, `nowrap`, `pre`, `pre-wrap`, `pre-line` |
| `word-break`, `overflow-wrap` | How long words break |
| `text-overflow: ellipsis` | "…" when text clips |
| `font-variant-numeric: tabular-nums` | Equal-width digits — perfect for tables and prices |
| `writing-mode` | `vertical-rl`, `vertical-lr` (CJK, vertical layouts) |

### Useful tricks

**One-line truncation:**
```css
white-space: nowrap;
overflow: hidden;
text-overflow: ellipsis;
```

**Multi-line clamp (still vendor-prefixed):**
```css
display: -webkit-box;
-webkit-line-clamp: 3;
-webkit-box-orient: vertical;
overflow: hidden;
```

---

## 8. Layout fundamentals (`06_layout/`)

Before flex/grid, you must understand `display`, `position`, and `overflow`.

### `display`

| Value | Behavior |
|--|--|
| `block` | New line, fills container width. |
| `inline` | Flows with text; `width`/`height`/vertical-margin **ignored**. |
| `inline-block` | Inline flow, but respects width/height/margins. |
| `none` | Removed from rendering AND the accessibility tree. |
| `contents` | Removes the element's box but keeps its children in flow (clever flex/grid trick). |
| `flow-root` | Same as block, but creates a new block formatting context — **stops margin collapse** and **contains floats**. |
| `flex`, `grid` | See Phases 7–8. |

### Hiding — three subtly different ways

| Method | Visible? | Takes space? | Clickable? | In AT? |
|--|--|--|--|--|
| `display: none` | no | no | no | no |
| `visibility: hidden` | no | **yes** | no | no |
| `opacity: 0` | no | yes | **yes** (add `pointer-events: none` to fix) | yes |

### `position`

| Value | Meaning |
|--|--|
| `static` | Default — in flow, can't be offset. |
| `relative` | In flow, but `top/right/bottom/left` shift visually. Crucially, becomes a **containing block** for `absolute` descendants. |
| `absolute` | Removed from flow. Anchored to nearest **positioned** ancestor (or the viewport if none). |
| `fixed` | Removed from flow. Pinned to the **viewport** — scrolls with the page only if its containing block is transformed. |
| `sticky` | In flow, behaves like `relative` until a scroll threshold is crossed, then "sticks" like `fixed`. Needs a scrollable ancestor. |

### `z-index` and stacking contexts

`z-index` only works on **positioned** elements (or flex/grid items).

The classic gotcha:

> An element with `z-index: 9999` inside a parent with `z-index: 1` will **never** beat a sibling of that parent with `z-index: 2`.

Why? Each parent forms a **stacking context**, and z-index is compared *only inside the same context*. New stacking contexts are created by:
- `position: relative/absolute/fixed/sticky` + any non-`auto` `z-index`.
- `opacity < 1`.
- `transform`, `filter`, `will-change`, `isolation: isolate`.

When in doubt, use **DevTools** → 3D layers panel to visualize stacking.

### `overflow`

| Value | Behavior |
|--|--|
| `visible` | Default — content spills past the box. |
| `hidden` | Clip; no scrollbar. |
| `scroll` | Scrollbars **always**. |
| `auto` | Scrollbars **only when needed** (preferred). |
| `clip` | Like hidden but lighter (no scroll context). |

Per-axis: `overflow-x`, `overflow-y`. Setting overflow on an element **establishes a new block formatting context** — so it also stops margin collapse and contains floats.

### Float (legacy)

`float: left/right` pulls an element out of flow and lets surrounding text wrap around it. Once *the* layout tool, now mostly used for **text wrap around figures**. The classic clearfix is replaced by `display: flow-root` on the container.

---

## 9. Flexbox — 1-D layout (`07_flexbox/`)

Flexbox lays out items along **one axis**. Think rows of nav links, columns of cards in a sidebar — anything that's a line.

```
flex-direction: row    →   main axis ─────────►
                           cross axis │
                                      ▼
```

### Container properties (set on the parent)

| Property | What it does |
|--|--|
| `display: flex` (or `inline-flex`) | Activates flex layout |
| `flex-direction` | `row` (default), `row-reverse`, `column`, `column-reverse` — sets the main axis |
| `flex-wrap` | `nowrap` (default), `wrap`, `wrap-reverse` |
| `gap` | Space between items (**no margin hacks**) |
| `justify-content` | Alignment along **main axis**: `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly` |
| `align-items` | Alignment along **cross axis** (single line): `stretch`, `flex-start`, `flex-end`, `center`, `baseline` |
| `align-content` | Alignment of **wrapped lines** along the cross axis |

### Item properties (set on the children)

| Property | What it does |
|--|--|
| `flex-grow` | Share of *leftover* space (`0` = none, `1` = equal share, `2` = double) |
| `flex-shrink` | How much to shrink when space is tight (`1` default, `0` = rigid) |
| `flex-basis` | Starting size before grow/shrink (`auto`, `0`, `200px`) |
| `flex` | Shorthand: `flex: 1` = `1 1 0` (fill equally), `flex: auto` = `1 1 auto`, `flex: none` = `0 0 auto` |
| `align-self` | Override the container's `align-items` for this one item |
| `order` | Visual reorder. **DOM is unchanged** — beware of breaking keyboard tab order |

### Iconic patterns

```css
/* Center anything */
.box { display: flex; justify-content: center; align-items: center; }

/* Push the last item right */
.nav { display: flex; gap: 1rem; }
.nav .login { margin-left: auto; }

/* Sticky footer */
body  { min-height: 100vh; display: flex; flex-direction: column; }
main  { flex: 1; }   /* takes leftover height; footer sits at bottom */

/* Sidebar + main */
.shell    { display: flex; }
.sidebar  { flex: 0 0 240px; }   /* fixed width */
.content  { flex: 1; }            /* fills the rest */
```

---

## 10. Grid — 2-D layout (`08_grid/`)

Grid lays out items along **two axes** simultaneously — rows *and* columns. Think page layouts, photo galleries, magazines.

### Defining tracks

```css
.grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;            /* three columns, ratios */
  grid-template-rows: auto 1fr auto;              /* header / body / footer */
  gap: 1rem;
}
```

Track-size values:
- **`fr`** — fraction of *leftover* space (the magic of grid).
- `repeat(3, 1fr)` — short for `1fr 1fr 1fr`.
- `minmax(200px, 1fr)` — minimum 200px, expand to fill.
- `auto` — size to content.

### The responsive one-liner

```css
grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
```

Translation: "fit as many ≥220px columns as the screen allows; share leftover space." This single line replaces dozens of media queries.

`auto-fit` collapses empty columns; `auto-fill` keeps them.

### Placing items

By line number:
```css
.hero { grid-column: 1 / 3; }     /* lines 1 → 3 = spans 2 columns */
.full { grid-column: 1 / -1; }    /* full width, line 1 to last */
.tall { grid-row: span 2; }       /* span 2 tracks from auto position */
```

By **named areas** (the readable approach):

```css
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "side   main"
    "foot   foot";
  grid-template-columns: 200px 1fr;
}
.layout > .h { grid-area: header; }
.layout > .s { grid-area: side; }
.layout > .m { grid-area: main; }
.layout > .f { grid-area: foot; }
```

### Alignment

Grid has the same six properties as flex, in two pairs:

| Move the *items* in their cells | Move the whole grid in its container |
|--|--|
| `justify-items`, `align-items` | `justify-content`, `align-content` |
| `place-items` (shorthand) | `place-content` (shorthand) |

Per-item: `justify-self`, `align-self`, `place-self`.

### Patterns to know

- **RAM grid** — `repeat(auto-fit, minmax(220px, 1fr))`. Instant responsive.
- **Holy Grail** — header / sidebar / main / footer via named areas, collapse to single column on mobile.
- **Magazine / dense packing** — `grid-auto-flow: dense` lets later items fill earlier holes.
- **Overlap** — assign multiple items to the same cell (`grid-area: 1 / 1`) and stack them with `z-index`.

---

## 11. Responsive design (`09_responsive/`)

### Units

| Unit | Relative to | Notes |
|--|--|--|
| `px` | Absolute | Pixel, the device-independent unit since high-DPI screens. |
| `em` | Parent's font-size | Compounds when nested. |
| `rem` | Root (`<html>`) font-size | Predictable, the typography workhorse. |
| `ch` | Width of "0" character | `max-width: 65ch` for readable line lengths. |
| `lh` | Line-height of the element | Spacing that tracks typography. |
| `%` | Parent's *width* (even for vertical paddings/margins!) | Surprising and important. |
| `vw`, `vh` | Viewport width / height | `100vw`/`100vh` = full screen. |
| `vmin`, `vmax` | The smaller/larger of vw/vh | Square hero shapes. |
| `dvh`, `svh`, `lvh` | Dynamic / small / large vh | Account for mobile browser bars showing/hiding. |
| `fr` | Grid leftover space | Grid only. |

### Math functions

```css
width: calc(100% - 2rem);
font-size: clamp(1rem, 2vw + 0.5rem, 2rem);   /* floor, fluid, ceiling */
width: min(100%, 60ch);                        /* whichever is smaller */
padding: max(1rem, 2vw);                       /* whichever is larger */
```

`clamp()` is the modern way to do **fluid typography** — `font-size` smoothly scales between mobile and desktop *without media queries*.

### Media queries

```css
@media (min-width: 600px) { ... }
@media (max-width: 599px) and (orientation: portrait) { ... }
@media (prefers-color-scheme: dark) { ... }
@media (prefers-reduced-motion: reduce) { ... }
@media (hover: none) { ... }   /* touch-only devices */
@media print { ... }
```

Combine with `and`; OR with comma. `not` inverts.

### Mobile-first

Write base styles for the smallest screen, then add `min-width` queries to upgrade:

```css
.grid { grid-template-columns: 1fr; }                                  /* phone */
@media (min-width: 600px)  { .grid { grid-template-columns: repeat(2, 1fr); } }
@media (min-width: 900px)  { .grid { grid-template-columns: repeat(3, 1fr); } }
```

Why? The base styles are simplest, additions are predictable, and CSS without a media-query default works on every device.

### Modern responsive — no breakpoints needed

- **`clamp()`** — fluid sizing.
- **`repeat(auto-fit, minmax(...))`** — auto-flowing columns.
- **`aspect-ratio: 16 / 9`** — keep proportions without padding-bottom hacks.
- **`@container card (min-width: 400px) { ... }`** — respond to the **parent's** width, not the viewport. Set `container-type: inline-size` on the parent. Component-level responsiveness — finally.
- **`prefers-color-scheme: dark`** — free dark mode.
- **`prefers-reduced-motion: reduce`** — respect motion-sensitive users.

---

## 12. Animations (`10_animations/`)

Three tools, escalating in power: **transitions** → **transforms** → **keyframes**.

### Transitions — interpolate between two states

```css
.btn          { background: blue; transition: background 0.25s ease; }
.btn:hover    { background: navy; }
```

`transition: property duration timing-function delay`.

Timing functions:
- `ease` — slow-fast-slow (default).
- `linear` — constant speed.
- `ease-in` — slow-fast (entering).
- `ease-out` — fast-slow (exiting; **feels best for UI**).
- `ease-in-out` — slow-fast-slow.
- `cubic-bezier(.2, .8, .4, 1)` — custom curves.

Multiple properties: comma-separated, or `transition: all .2s ease` (simpler but can over-trigger).

Not animatable: `display`, `font-family`, anything discrete.

### Transforms — move/scale/rotate without re-layout

```css
.card { transition: transform .2s; }
.card:hover { transform: translateY(-4px) scale(1.02); }
```

2D: `translate(x, y)`, `translateX/Y`, `scale(x, y)`, `rotate(deg)`, `skew(x, y)`.
3D: `rotateX/Y/Z`, `translate3d`, `scale3d`. Needs `perspective` on the parent.

**Combinations apply right-to-left**: `transform: translateY(-10px) rotate(15deg) scale(1.1)` first scales, then rotates, then translates.

`transform-origin` sets the pivot (default `50% 50%`).

**Why transforms matter**: they are **GPU-accelerated** and don't trigger layout/paint — they're the cheapest way to animate. Animating `width`, `height`, `top`, `left` *does* trigger layout (slow). When you can express the same change with `transform`, do.

### Keyframes — multi-stage choreography

```css
@keyframes pulse {
  0%, 100% { opacity: 0.3; }
  50%      { opacity: 1; }
}

.dot {
  animation: pulse 1.2s ease-in-out infinite;
}
```

The `animation` shorthand: `name duration timing-function delay iteration-count direction fill-mode play-state`.

Key sub-properties:
- `animation-iteration-count` — `1`, `3`, `infinite`.
- `animation-direction` — `normal`, `reverse`, `alternate`, `alternate-reverse`.
- `animation-fill-mode` — `none`, `forwards` (keep final state — the most useful), `backwards` (apply initial state during delay), `both`.
- `animation-play-state` — `running`, `paused`.

### Always respect reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

Vestibular disorders are real — and so is the user setting that says "I don't want motion."

---

## 13. Modern CSS (`11_modern/`)

### Custom properties (CSS variables)

```css
:root {
  --brand: #6366f1;
  --space: 1rem;
  --radius: 0.5rem;
}

.card {
  background: var(--brand);
  padding: var(--space);
  border-radius: var(--radius, 4px);  /* fallback if undefined */
}

body[data-theme="dark"] { --brand: #76d6ff; }
```

Unlike Sass variables, CSS custom properties:
- **Inherit** through the cascade — define at `:root` for globals, override per component.
- Are **live at runtime** — JavaScript can change them: `el.style.setProperty('--brand', 'red')`.
- Power **theming** with a single attribute swap.

### `:is()`, `:where()`, and `:has()`

```css
/* :is — group selectors. Specificity = highest inside (matters!) */
:is(h1, h2, h3) { font-family: 'Inter'; }

/* :where — same, but ZERO specificity. Perfect for resets. */
:where(button, input, select) { font: inherit; }

/* :has — finally, a parent selector */
.card:has(.unread)            { border-color: orange; }
form:has(input:invalid)        { background: #fdecea; }
li:has(+ li:hover)             { opacity: 0.5; }   /* style based on next sibling's state */
```

`:has()` enables CSS-only patterns that previously required JavaScript: dropdowns that style their parent when open, forms that highlight on validation errors, lists that fade un-hovered items.

### Native nesting (2023+)

```css
.card {
  padding: 1rem;

  & h2     { color: navy; }
  &:hover  { background: #fafafa; }
  & .badge { color: white; }

  @media (min-width: 600px) { padding: 2rem; }
}
```

Reduces repetition without a preprocessor. The `&` is the parent selector reference. (You can usually omit it before a tag/class, but it's required before pseudo-classes like `&:hover`.)

### Logical properties

Instead of physical directions (`left`, `right`, `top`, `bottom`), write *logical* ones tied to the writing direction:

| Physical | Logical |
|--|--|
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `padding-top` | `padding-block-start` |
| `width` | `inline-size` |
| `height` | `block-size` |
| `border-left` | `border-inline-start` |

Switch `dir="rtl"` and the same CSS works — Arabic, Hebrew, Japanese vertical, all from a single stylesheet. Future-proof for i18n.

---

## 14. Capstone projects (`12_projects/`)

Each project deliberately reuses concepts from earlier phases — they're a recall exercise.

| # | Project | Concepts exercised |
|--|--|--|
| 34 | **Animated buttons** | Transitions, transforms, `::before`/`::after`, gradients, keyframes, layered shadows |
| 35 | **Pure-CSS modal** | `:target` pseudo-class drives open/close (zero JS), `position: fixed`, `backdrop-filter`, transform entrance |
| 36 | **Pricing table** | `auto-fit` grid, custom properties for theming, featured-tier badge via `::before`, flex column with `flex: 1` for button alignment |
| 37 | **Image gallery** | Magazine grid (span 2 cols/rows on featured items), `grid-auto-flow: dense`, `aspect-ratio`, hover overlays with `object-fit: cover` |
| 38 | **Animated hero** | Animated gradient (`background-size: 400%` + keyframe), floating shapes, glassmorphism (`backdrop-filter: blur`), staggered fade-up via `animation-delay`, `clamp()` typography |

The general rule: **modern UI polish is a stack of small CSS techniques layered together**, not a single trick.

---

## 15. Cross-cutting principles

These show up in every phase. Internalize them and CSS feels predictable.

1. **The cascade is the system.** Most "weird CSS" is a cascade misunderstanding — use DevTools' Computed panel to see the *winning* value, and the Styles panel to see every rule that matched.
2. **Reset `box-sizing` to `border-box` once, globally.** It's free sanity.
3. **Animate `transform` and `opacity` only.** Anything else triggers layout or paint and stutters.
4. **Mobile-first, content-first.** Build the smallest screen first; add complexity at larger breakpoints.
5. **Prefer modern over historical.** Flex/grid over float. `gap` over margin hacks. `clamp()` over breakpoint chains. `:has()` over JS toggles.
6. **Respect user preferences.** `prefers-color-scheme`, `prefers-reduced-motion`, `:focus-visible`. The web is everyone's.
7. **Specificity wars are a smell.** When you need `!important` or `#id #id .class .class`, the architecture is wrong.

---

## 16. Lesson index

```
PHASE 1 — Foundations (01_foundations/)
Step 1   01_three_ways.html              Inline / internal / external
Step 2   02_syntax_and_cascade.html      Syntax, cascade, !important
Step 3   03_specificity_inheritance.html Specificity scoring, inheritance

PHASE 2 — Selectors (02_selectors/)
Step 4   01_basic_and_attribute.html     Tag, class, id, group, attribute matchers
Step 5   02_combinators.html             Descendant, child, adjacent & general sibling
Step 6   03_pseudo_classes.html          :hover, :focus, :nth-child, :not, :is, :where
Step 7   04_pseudo_elements.html         ::before, ::after, ::first-letter, ::placeholder, ::marker

PHASE 3 — Box Model (03_box_model/)
Step 8   01_box_model.html               Content, padding, border, margin, box-sizing
Step 9   02_margin_collapsing.html       Vertical margin collapsing, negative margins

PHASE 4 — Visual (04_visual/)
Step 10  01_colors.html                  Hex, rgb, hsl, oklch, currentColor, theming
Step 11  02_backgrounds_gradients.html   Backgrounds, linear/radial/conic gradients
Step 12  03_borders_radius_outline.html  Borders, border-radius, outline, focus rings
Step 13  04_shadows.html                 box-shadow & text-shadow

PHASE 5 — Typography (05_typography/)
Step 14  01_fonts.html                   Font stacks, web fonts, @font-face, font-display
Step 15  02_text_styling.html            Align, transform, decoration, line-height, line-clamp

PHASE 6 — Layout (06_layout/)
Step 16  01_display.html                 block, inline, inline-block, none, hide patterns
Step 17  02_position_zindex.html         relative, absolute, fixed, sticky, stacking contexts
Step 18  03_overflow_float.html          overflow, float (text wrap), scroll-snap

PHASE 7 — Flexbox (07_flexbox/)
Step 19  01_container.html               direction, justify-content, align-items, gap, wrap
Step 20  02_items.html                   flex-grow/shrink/basis, align-self, order
Step 21  03_patterns.html                Centering, navbar, cards, sticky footer

PHASE 8 — Grid (08_grid/)
Step 22  01_grid_basics.html             Template columns, fr, repeat, minmax, auto-fit
Step 23  02_placement_areas.html         Line-based placement, span, named areas, place-items
Step 24  03_responsive_patterns.html     RAM grid, Holy Grail, magazine, 12-col, overlap

PHASE 9 — Responsive (09_responsive/)
Step 25  01_units.html                   px, em, rem, ch, vw/vh, fr, calc, min, max, clamp
Step 26  02_media_queries.html           Mobile-first, dark mode, reduced motion
Step 27  03_modern_responsive.html       clamp(), aspect-ratio, container queries

PHASE 10 — Animations (10_animations/)
Step 28  01_transitions.html             Transitions, timing functions, delays
Step 29  02_transforms.html              translate/scale/rotate/skew, 3D flip card
Step 30  03_keyframes.html               @keyframes, bounce, spin, shimmer, fade-up

PHASE 11 — Modern CSS (11_modern/)
Step 31  01_custom_properties.html       CSS variables, design tokens, theming
Step 32  02_is_where_has.html            :is(), :where(), :has() — the parent selector
Step 33  03_nesting_logical.html         Native nesting, logical properties (RTL-ready)

PHASE 12 — Projects (12_projects/)
Step 34  01_animated_buttons.html        Eight hover-animated buttons
Step 35  02_pure_css_modal.html          Modal driven by :target — zero JavaScript
Step 36  03_pricing_table.html           Three-tier pricing with featured plan
Step 37  04_image_gallery.html           Magazine-style grid with hover overlays
Step 38  05_animated_hero.html           Animated gradient hero with floating shapes
```

---

## 17. How to use this guide

1. **Read the theory section** for the phase you're on — this README.
2. **Open the lesson** in a browser (just double-click; no build, no server).
3. **Read the inline comments** — every concept is explained at point-of-use.
4. **Edit and break things.** Change values, remove rules, reload. Notice what shifted.
5. **Use DevTools.** F12 → Elements → Computed/Styles. The Computed tab shows the *winning* value for each property; the Styles tab shows every matching rule in cascade order.

---
