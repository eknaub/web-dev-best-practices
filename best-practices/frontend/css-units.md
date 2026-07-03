# CSS Units: When to Use em, rem, %, px, and More

Practical rules for choosing CSS units based on accessibility, responsiveness, and maintainability.

## Navigation Index

- [Core Rule of Thumb](#core-rule-of-thumb)
- [Quick Decision Table](#quick-decision-table)
- [Unit-by-Unit Guidance](#unit-by-unit-guidance)
  - [rem](#rem)
  - [em](#em)
  - [Percent (%)](#percent-)
  - [px](#px)
  - [vw and vh](#vw-and-vh)
  - [ch](#ch)
  - [fr (Grid)](#fr-grid)
- [What to Use for Common CSS Properties](#what-to-use-for-common-css-properties)
- [Recommended Baseline Setup](#recommended-baseline-setup)
- [Common Mistakes to Avoid](#common-mistakes-to-avoid)

---

## Core Rule of Thumb

- Use `rem` as your default for type sizes, spacing, and scalable component dimensions.
- Use `em` when a value should scale with the component's own font size.
- Use `%` when sizing should be relative to a parent/container.
- Use `px` for exact visual details where proportional scaling is not desired.

## Quick Decision Table

| Situation                              | Best Unit             | Why                                          |
| -------------------------------------- | --------------------- | -------------------------------------------- |
| Global font sizes and spacing scale    | `rem`                 | Respects root font size and user preferences |
| Component-relative padding/gaps        | `em`                  | Scales with local component text             |
| Fluid widths in a layout               | `%`                   | Responds to parent/container width           |
| Borders and one-off pixel precision    | `px`                  | Predictable, exact rendering                 |
| Full-screen hero/min viewport sections | `vw`, `vh` (or `dvh`) | Tied to viewport dimensions                  |
| Readable text measure (line length)    | `ch`                  | Relative to character width                  |
| Grid proportional columns              | `fr`                  | Native ratio-based track sizing              |

## Unit-by-Unit Guidance

### rem

Use `rem` for:

- Body text and heading scale.
- Spacing system (`margin`, `padding`, `gap`).
- Border radius and consistent component dimensions.

Why:

- `rem` is based on the root font size, so scaling is consistent across the app.
- It improves accessibility when users increase browser default font size.

```css
:root {
  font-size: 100%; /* usually 16px */
}

body {
  font-size: 1rem;
}

.card {
  padding: 1rem;
  border-radius: 0.5rem;
  gap: 0.75rem;
}
```

### em

Use `em` for:

- Internal spacing that should grow/shrink with local text size.
- Component variants like `button--sm`, `button--lg`.

Why:

- `em` is relative to the element's own font size (or parent font size in `font-size` itself), which makes component-local scaling easy.

```css
.button {
  font-size: 1rem;
  padding: 0.6em 1em;
}

.button--lg {
  font-size: 1.25rem;
}
```

### Percent (%)

Use `%` for:

- Widths that should adapt to parent containers.
- Heights only when the parent has a defined height.
- Responsive media sizing.

Why:

- `%` expresses container relationships directly, so layouts stay fluid.

```css
.container {
  max-width: 72rem;
  width: min(100% - 2rem, 72rem);
  margin-inline: auto;
}

img {
  max-width: 100%;
  height: auto;
}
```

### px

Use `px` for:

- Borders (`1px`, `2px`) and subtle shadows.
- Hairline separators and exact icon alignment tweaks.
- Cases where precision is more important than scaling.

Why:

- Pixels are explicit and predictable.

```css
.input {
  border: 1px solid #c5c5c5;
}

.separator {
  height: 1px;
}
```

### vw and vh

Use `vw`/`vh` for:

- Viewport-based hero sections and typography accents.
- Full-height panels (`100dvh` is safer on mobile than `100vh`).

Why:

- They scale with the viewport, not parent containers.

```css
.hero {
  min-height: 100dvh;
  padding-block: clamp(2rem, 6vh, 6rem);
}
```

### ch

Use `ch` for:

- Constraining paragraph width for readability.

Why:

- `ch` approximates character width, making line-length control easier.

```css
.prose {
  max-width: 65ch;
}
```

### fr (Grid)

Use `fr` for:

- Proportional columns and rows in CSS Grid.

Why:

- It expresses ratios directly and avoids fragile percentage math.

```css
.grid {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 1rem;
}
```

## What to Use for Common CSS Properties

- `font-size`: usually `rem`; sometimes `clamp()` with `rem` and `vw`.
- `line-height`: unitless for inheritance (`1.4`, `1.6`) in most cases.
- `margin`/`padding`/`gap`: mostly `rem`, optionally `em` for component-relative scaling.
- `width`: `%`, `min()`, `max()`, `clamp()`, and layout units like `fr`.
- `height`: `auto` first; `%` only with known parent height; `dvh` for viewport sections.
- `border-width`: `px`.
- `border-radius`: `rem` for scalable UI, `px` for exact corner control.

## Recommended Baseline Setup

```css
:root {
  font-size: 100%;
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --radius-1: 0.375rem;
}

body {
  font-size: 1rem;
  line-height: 1.5;
}
```

## Common Mistakes to Avoid

- Using only `px` for typography and spacing, which can reduce accessibility.
- Using `em` everywhere and accidentally compounding scale through nesting.
- Using `%` height without a defined parent height and expecting it to work.
- Using `vh` on mobile without checking browser UI behavior; prefer `dvh` when possible.
- Setting `line-height` in `px` for body text, which can break scaling.
