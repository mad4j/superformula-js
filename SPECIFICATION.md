# RFC-001: Superformula Interactive Visualizer — Specification

| Field        | Value                                |
|--------------|--------------------------------------|
| RFC          | 001                                  |
| Title        | Superformula Interactive Visualizer  |
| Status       | Draft                                |
| Author       | mad4j                                |
| Created      | 2026-03-11                           |
| License      | MIT                                  |

---

## Table of Contents

1. [Overview](#1-overview)
2. [Terminology](#2-terminology)
3. [Mathematical Foundation](#3-mathematical-foundation)
4. [Architecture](#4-architecture)
5. [Canvas & Rendering](#5-canvas--rendering)
6. [Parameter Controls](#6-parameter-controls)
7. [Colour System](#7-colour-system)
8. [Randomise Feature](#8-randomise-feature)
9. [Panel Collapse](#9-panel-collapse)
10. [Responsiveness](#10-responsiveness)
11. [Accessibility](#11-accessibility)
12. [Visual Style](#12-visual-style)
13. [Saved Configurations Gallery](#13-saved-configurations-gallery)
14. [Requirements Index](#14-requirements-index)

---

## 1. Overview

This document specifies the design and behavioural requirements for **superformula-js**: a
single-file, zero-dependency, browser-based interactive visualizer of the Gielis Superformula.
The application renders the curve on an HTML5 `<canvas>` element and exposes six formula
parameters as range sliders, together with stroke colour, fill colour, filled/outline toggle, and a
randomise button.

**REQ-001** The entire application MUST be delivered as a single self-contained `index.html` file
with no external runtime dependencies, no build step, and no server requirement.

**REQ-002** The application MUST run correctly in any modern evergreen browser (Chrome, Firefox,
Safari, Edge) when `index.html` is opened directly from the local file system.

---

## 2. Terminology

| Term | Meaning |
|------|---------|
| *superformula* | The polar-coordinate curve defined in §3. |
| *parameter* | One of the six scalar values `m`, `a`, `b`, `n₁`, `n₂`, `n₃`. |
| *canvas* | The HTML5 `<canvas>` element used for rendering. |
| *controls panel* | The overlay at the bottom of the viewport containing all interactive widgets. |
| *stroke colour* | The colour used to draw the outline of the curve. |
| *fill colour* | The colour used to flood-fill the interior of the curve. |
| *DPR* | Device Pixel Ratio (`window.devicePixelRatio`). |

---

## 3. Mathematical Foundation

### 3.1 The Superformula

The Superformula, introduced by Johan Gielis (2003), is expressed in polar coordinates as:

```
r(φ) = [ |cos(m·φ/4) / a|^n2  +  |sin(m·φ/4) / b|^n3 ] ^ (-1/n1)
```

where:
- `φ` (phi) is the angle in radians, ranging over `[0, 2π)`.
- `r(φ)` is the radial distance at angle `φ`.
- `m`, `a`, `b`, `n₁`, `n₂`, `n₃` are the six real-valued parameters defined in §6.

**REQ-003** The implementation MUST compute `r(φ)` exactly as shown above, using
`Math.abs`, `Math.cos`, `Math.sin`, `Math.pow` (or equivalent) without approximation shortcuts.

**REQ-004** When the intermediate value `v = |cos(m·φ/4)/a|^n2 + |sin(m·φ/4)/b|^n3` equals
zero, `r(φ)` MUST be treated as `0` (instead of `Infinity` or `NaN`).

### 3.2 Curve Normalisation

**REQ-005** Before drawing, the implementation MUST compute the maximum radial value
`maxR = max{ r(φ) | φ ∈ [0, 2π) }` over all sample angles.

**REQ-006** If `maxR` equals `0`, drawing MUST be skipped for that frame.

**REQ-007** The shape MUST be scaled so that its maximum radius exactly equals the available
drawing radius (see REQ-020), using the factor `scale = drawingRadius / maxR`.

---

## 4. Architecture

**REQ-008** All application logic MUST be contained in a single inline `<script>` block inside
`index.html`, written in plain JavaScript (ES2015+) with `'use strict'` mode enabled.

**REQ-009** The application MUST NOT use any external JavaScript libraries, CSS frameworks, or
Web Component polyfills.

**REQ-010** The application structure MUST consist of exactly three top-level HTML elements
inside `<body>`:
1. `#canvas-container` — a fixed, full-viewport `<div>` that centers the `<canvas>` element.
2. `#canvas` — the `<canvas>` element for rendering.
3. `#controls` — a fixed panel anchored to the bottom edge of the viewport.

---

## 5. Canvas & Rendering

### 5.1 Canvas Setup

**REQ-011** The canvas context MUST be obtained with `{ alpha: false }` to improve compositing
performance.

**REQ-012** The canvas physical size (in device pixels) MUST be computed as
`Math.round(side × DPR)` for both width and height, where `side` is the CSS size of the canvas in
logical pixels (see REQ-021).

**REQ-013** The canvas CSS size MUST equal `side × side` (logical pixels), making it a square.

### 5.2 Drawing Procedure

**REQ-014** Each draw call MUST:
1. Clear the canvas by filling the entire area with the background colour `#f8fafc`.
2. Compute parameters from the current slider values.
3. Sample `r(φ)` at `1 201` equally-spaced angles (`i = 0 … 1200`) over `[0, 2π)`:
   `φ_i = 2π · i / 1200`.
4. Determine `maxR` from those samples (REQ-005 / REQ-006).
5. Construct a single closed `Path2D` / canvas path from the 1 201 scaled Cartesian points
   `(cx + r·cos(φ), cy + r·sin(φ))`, where `cx = W/2`, `cy = H/2`.
6. If the *Filled* toggle is checked, flood-fill the path with the current fill colour.
7. Always stroke the path with the current stroke colour.
8. Draw subtle reference axes (REQ-024).

**REQ-015** The stroke width MUST be `max(1.5, W × 0.003)` device pixels, where `W` is the
physical canvas width.

**REQ-016** The stroke line join style MUST be `'round'`.

**REQ-017** The draw function MUST be called:
- After every slider `input` event.
- After every colour selection.
- After toggling the *Filled* checkbox.
- After every randomise action.
- After every resize / panel-collapse event.

### 5.3 Reference Axes

**REQ-018** Two dashed reference lines MUST be drawn through the canvas centre after the curve:
- A horizontal line from `(cx − r·1.05, cy)` to `(cx + r·1.05, cy)`.
- A vertical line from `(cx, cy − r·1.05)` to `(cx, cy + r·1.05)`.

**REQ-019** The reference lines MUST use stroke style `rgba(0,0,0,0.08)`, line width `1`, and
dash pattern `[4, 6]`.

---

## 6. Parameter Controls

### 6.1 Slider Layout

**REQ-020** The controls panel MUST contain exactly six slider groups arranged in a CSS Grid
with three columns on viewports wider than 480 px CSS and two columns on viewports
`≤ 480 px` CSS.

**REQ-021** The drawing area (canvas side length) MUST be the minimum of:
- The viewport width (`window.innerWidth`).
- The available height `window.innerHeight − panelH`, where `panelH` is the height of the
  controls panel (or `44 px` when collapsed).

### 6.2 Parameter Definitions

Each slider MUST conform to the following specification:

The HTML element `id` uses plain ASCII (e.g. `sl-n1`); the visible label uses the Unicode
subscript glyph (e.g. `n₁`) for typographic clarity. Both refer to the same parameter.

| Element ID | Display label | `min` | `max` | `step` | Default | Display format |
|------------|---------------|-------|-------|--------|---------|----------------|
| `sl-m`  | m  | 1    | 20  | 1    | 6   | integer string |
| `sl-a`  | a  | 0.1  | 5   | 0.05 | 1   | 2 decimal places |
| `sl-b`  | b  | 0.1  | 5   | 0.05 | 1   | 2 decimal places |
| `sl-n1` | n₁ | 0.1  | 20  | 0.1  | 1   | 2 decimal places |
| `sl-n2` | n₂ | 0.1  | 20  | 0.1  | 1   | 2 decimal places |
| `sl-n3` | n₃ | 0.1  | 20  | 0.1  | 1   | 2 decimal places |

**REQ-022** Each slider group MUST display the parameter symbol (italic, accent colour) on the
left and the current numeric value on the right of the label row.

**REQ-023** Each slider MUST display a filled-track effect: the portion of the track between the
minimum and the thumb position MUST appear in the accent colour (`--accent`), and the
remainder in a muted colour. The percentage MUST be recomputed on every `input` event and
stored as a CSS custom property `--pct` on the `<input>` element.

---

## 7. Colour System

### 7.1 Palette

**REQ-024** A fixed palette of exactly **12 colours** MUST be provided, arranged in two rows of
six within the colour picker popover:

| Row | Colours (hex) |
|-----|---------------|
| Dark tones  | `#0f172a` `#1e3a5f` `#1e40af` `#374151` `#1c1917` `#164e63` |
| Light tones | `#bfdbfe` `#e0e7ff` `#e2e8f0` `#d1fae5` `#fef9c3` `#fce7f3` |

### 7.2 Stroke Colour

**REQ-025** The initial stroke colour MUST be `#1e40af`.

**REQ-026** A "Stroke" button MUST open a colour picker popover showing all 12 palette swatches.
Clicking a swatch MUST set the stroke colour to that value, update the swatch preview on the
button, close the popover, and trigger a redraw.

### 7.3 Fill Colour

**REQ-027** The initial fill colour MUST be `#bfdbfe`.

**REQ-028** A "Fill" button MUST open a colour picker popover following the same behaviour as
REQ-026 but for the fill colour.

### 7.4 Filled Toggle

**REQ-029** A toggle switch labelled "Filled" MUST control whether the curve interior is filled.
The toggle MUST be checked (filled) by default.

**REQ-030** When the toggle is unchecked, only the stroke is drawn; no fill operation is
performed.

### 7.5 Popover Behaviour

**REQ-031** Only one colour picker popover MAY be open at a time. Opening a second popover
MUST close the first.

**REQ-032** Clicking anywhere outside a colour picker popover MUST close all open popovers.

**REQ-033** Each palette swatch MUST reflect its selected state visually (a ring highlight) and via
`aria-selected="true"`.

---

## 8. Randomise Feature

**REQ-034** A "RAND" button (Unicode die symbol `⚄` followed by the text `RAND`) MUST be
present in the extras row of the controls panel.

**REQ-035** Clicking the RAND button MUST set each of the six parameters to an independent,
uniformly random value within its allowed range (`min`…`max`), snapped to the nearest `step`
increment.

**REQ-036** The random value for each parameter MUST be calculated as:

```
steps  = floor((max − min) / step)
raw    = min + round(random() × steps) × step
value  = raw.toFixed(decimals)         // decimals = 0 when step ≥ 1
```

**REQ-037** After randomisation, all slider positions, value labels, track-fill percentages, and the
canvas MUST be updated in a single synchronous pass before the next animation frame.

---

## 9. Panel Collapse

**REQ-038** The controls panel MUST have a "Superformula Parameters" toggle button at the top.
Clicking it MUST toggle the CSS class `collapsed` on the panel element.

**REQ-039** When `collapsed`, the panel MUST slide out of view via CSS `transform:
translateY(calc(100% - 44px))`, leaving only the 44 px toggle-button strip visible.

**REQ-040** The chevron icon inside the toggle button MUST rotate 180° when the panel is
collapsed, achieved via CSS `transform: rotate(180deg)`.

**REQ-041** After every collapse/expand transition, the canvas MUST be resized and redrawn
(REQ-017 applies).

---

## 10. Responsiveness

**REQ-042** On every `window` `resize` event, the canvas MUST be resized and redrawn.
Resize events MUST be debounced using `requestAnimationFrame` (one pending RAF at most).

**REQ-043** The `#canvas-container` `bottom` CSS property MUST be dynamically updated to
match the controls panel height so the canvas never overlaps the panel.

**REQ-044** On viewports `≤ 480 px` wide, the slider grid MUST reflow to two columns
(REQ-020).

---

## 11. Accessibility

**REQ-045** The toggle button (`#toggle-btn`) MUST have `aria-label="Toggle controls"`.

**REQ-046** The colour picker buttons MUST have `aria-label` describing the target
(e.g. `"Stroke colour"`, `"Fill colour"`), `aria-haspopup="true"`, and `aria-expanded` set to
`"true"` / `"false"` based on popover visibility.

**REQ-047** Each colour palette container MUST have `role="listbox"` and a descriptive
`aria-label`.

**REQ-048** Each palette swatch MUST have `role="option"`, `aria-label` set to the hex colour
string, and `aria-selected` reflecting selection state.

**REQ-049** The RAND button MUST have `aria-label="Randomise parameters"`.

**REQ-050** All interactive controls MUST be keyboard-operable (native `<button>` and
`<input type="range">` elements satisfy this requirement intrinsically).

---

## 12. Visual Style

**REQ-051** The application MUST define the following CSS custom properties on `:root`:

| Property | Value |
|----------|-------|
| `--panel-bg` | `rgba(255, 255, 255, 0.92)` |
| `--accent` | `#2563eb` |
| `--accent-light` | `#3b82f6` |
| `--text` | `#1e293b` |
| `--text-muted` | `#64748b` |
| `--border` | `rgba(0, 0, 0, 0.12)` |
| `--thumb-size` | `22px` |
| `--track-height` | `4px` |

**REQ-052** The controls panel MUST use a frosted-glass effect via `backdrop-filter: blur(12px)`
and `background: var(--panel-bg)`.

**REQ-053** The page background colour MUST be `#f8fafc` (both `<html>`/`<body>` and the
canvas clear colour).

**REQ-054** The body MUST use `font-family: system-ui, -apple-system, sans-serif`.

**REQ-055** `overflow: hidden` MUST be applied to both `<html>` and `<body>` to prevent
scrollbars.

**REQ-056** Slider thumb and track MUST be fully custom-styled (native appearance suppressed via
`-webkit-appearance: none` / `appearance: none`) to match the accent colour scheme defined in
REQ-051.

**REQ-057** The RAND button MUST use `background: var(--accent)` normally, `var(--accent-light)`
on hover, and `#1d4ed8` on active.

---

## 13. Saved Configurations Gallery

### 13.1 Overview

**REQ-058** A "SAVE" button MUST be present in the extras row, positioned to the right of the
*Filled* toggle and to the left of the RAND button.

**REQ-059** Clicking the SAVE button MUST capture the complete current state — all eight parameter
values (`m`, `mx`, `my`, `a`, `b`, `n₁`, `n₂`, `n₃`), stroke colour, fill colour, and the
*Filled* toggle state — and prepend it to an in-memory gallery array.

**REQ-060** The gallery MUST retain at most **5** saved configurations. When a sixth item is saved,
the oldest entry MUST be discarded.

### 13.2 Gallery Display

**REQ-061** A gallery row MUST be rendered inside the `#controls` panel, below the extras row.
When the gallery is empty the row MUST be hidden; it MUST become visible as soon as the first
configuration is saved.

**REQ-062** Each saved configuration MUST be represented by a square thumbnail button (48 × 48 CSS
pixels). The thumbnail MUST display the superformula curve for that configuration, rendered with
the saved stroke colour, fill colour, and *Filled* state, on the background colour `#f8fafc`.

**REQ-063** Clicking a gallery thumbnail MUST restore all saved parameter values and colour
settings to the interactive controls and trigger a full redraw of the main canvas, exactly as if
the user had set those controls manually.

---

## 14. Requirements Index

| ID | Section | Summary |
|----|---------|---------|
| REQ-001 | §1 | Single `index.html` file, no dependencies |
| REQ-002 | §1 | Runs in modern browsers from local filesystem |
| REQ-003 | §3.1 | Superformula computed correctly |
| REQ-004 | §3.1 | `r = 0` when intermediate value `v = 0` |
| REQ-005 | §3.2 | Compute `maxR` before drawing |
| REQ-006 | §3.2 | Skip draw when `maxR = 0` |
| REQ-007 | §3.2 | Normalise shape to fit drawing radius |
| REQ-008 | §4 | Single inline `<script>`, `'use strict'` |
| REQ-009 | §4 | No external libraries |
| REQ-010 | §4 | Three top-level body elements |
| REQ-011 | §5.1 | Canvas context `{ alpha: false }` |
| REQ-012 | §5.1 | Physical size = `round(side × DPR)` |
| REQ-013 | §5.1 | CSS size is a square |
| REQ-014 | §5.2 | Full drawing procedure |
| REQ-015 | §5.2 | Stroke width `max(1.5, W × 0.003)` |
| REQ-016 | §5.2 | Line join `'round'` |
| REQ-017 | §5.2 | Redraw triggers |
| REQ-018 | §5.3 | Two reference axes |
| REQ-019 | §5.3 | Axis style: dashed, 8% opacity |
| REQ-020 | §6.1 | 3-column / 2-column grid layout |
| REQ-021 | §6.1 | Square canvas sizing formula |
| REQ-022 | §6.2 | Parameter symbol + value in label row |
| REQ-023 | §6.2 | Filled track via `--pct` CSS property |
| REQ-024 | §7.1 | 12-colour fixed palette |
| REQ-025 | §7.2 | Initial stroke colour `#1e40af` |
| REQ-026 | §7.2 | Stroke colour picker behaviour |
| REQ-027 | §7.3 | Initial fill colour `#bfdbfe` |
| REQ-028 | §7.3 | Fill colour picker behaviour |
| REQ-029 | §7.4 | Filled toggle, default checked |
| REQ-030 | §7.4 | No fill when toggle unchecked |
| REQ-031 | §7.5 | One popover open at a time |
| REQ-032 | §7.5 | Click outside closes popovers |
| REQ-033 | §7.5 | Selected swatch visual + ARIA state |
| REQ-034 | §8 | RAND button with die symbol |
| REQ-035 | §8 | Randomise all six parameters |
| REQ-036 | §8 | Random value formula |
| REQ-037 | §8 | Synchronous UI update after randomise |
| REQ-038 | §9 | Toggle button collapses panel |
| REQ-039 | §9 | Collapsed transform |
| REQ-040 | §9 | Chevron rotation animation |
| REQ-041 | §9 | Resize + redraw after collapse |
| REQ-042 | §10 | Debounced resize via RAF |
| REQ-043 | §10 | `#canvas-container` bottom tracks panel |
| REQ-044 | §10 | 2-column grid at ≤ 480 px |
| REQ-045 | §11 | Toggle button ARIA label |
| REQ-046 | §11 | Colour button ARIA attributes |
| REQ-047 | §11 | Palette container `role="listbox"` |
| REQ-048 | §11 | Palette swatch ARIA attributes |
| REQ-049 | §11 | RAND button ARIA label |
| REQ-050 | §11 | Keyboard operability |
| REQ-051 | §12 | CSS custom properties |
| REQ-052 | §12 | Frosted-glass panel |
| REQ-053 | §12 | Background colour `#f8fafc` |
| REQ-054 | §12 | System font stack |
| REQ-055 | §12 | `overflow: hidden` on html/body |
| REQ-056 | §12 | Custom slider styling |
| REQ-057 | §12 | RAND button colour states |
| REQ-058 | §13 | SAVE button in extras row |
| REQ-059 | §13 | Capture and store current configuration on save |
| REQ-060 | §13 | Gallery retains at most 5 configurations |
| REQ-061 | §13 | Gallery row hidden when empty, visible when populated |
| REQ-062 | §13 | Thumbnail renders saved curve at 48 × 48 CSS pixels |
| REQ-063 | §13 | Clicking thumbnail restores configuration and redraws |

---

*End of RFC-001*
