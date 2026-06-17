# Frontend General

## Navigation Index

- [CSS and Layout Basics](#css-and-layout-basics)
  - [Display Types](#display-types)
  - [Overflow](#overflow)
  - [Positioning](#positioning)
  - [Centering](#centering)
  - [Spacing Shorthand](#spacing-shorthand)
  - [CSS Specificity](#css-specificity)
- [SOLID Principles](#solid-principles)
  - [Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
  - [Open/Closed Principle (OCP)](#openclosed-principle-ocp)
  - [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
  - [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
  - [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
- [Security Essentials](#security-essentials)
  - [XSS and Content Safety](#xss-and-content-safety)
  - [General Security Rules](#general-security-rules)
- [When to Use Nullish Coalescing vs Logical OR](#when-to-use-nullish-coalescing-vs-logical-or)
- [Truthy and Falsy Values](#truthy-and-falsy-values)
- [Error Handling Essentials](#error-handling-essentials)
  - [Error Boundary Strategy](#error-boundary-strategy)
  - [What Boundaries Don't Catch](#what-boundaries-dont-catch)
  - [Practical Handling Rules](#practical-handling-rules)
- [Accessibility Quick Wins](#accessibility-quick-wins)
  - [Text Color Contrast](#text-color-contrast)
  - [Alt Text for Images](#alt-text-for-images)
  - [ARIA Live Regions](#aria-live-regions)
  - [Hiding Content Accessibly](#hiding-content-accessibly)
  - [Better Links (Descriptive + Visible)](#better-links-descriptive--visible)
  - [Input Labels + Placeholder Usage](#input-labels--placeholder-usage)
  - [Fieldset and Legend for Grouped Controls](#fieldset-and-legend-for-grouped-controls)
  - [Landmark Regions](#landmark-regions)
  - [Use rem for Font Sizes](#use-rem-for-font-sizes)
  - [Heading Structure](#heading-structure)
- [Critical Accessibility Principles](#critical-accessibility-principles)
  - [Forms](#forms)
  - [Keyboard and Focus](#keyboard-and-focus)
  - [ARIA Usage](#aria-usage)
- [Form Handling Best Practices](#form-handling-best-practices)
- [API/Fetch Patterns](#apifetch-patterns)
- [Mirage JS Mock Server](#mirage-js-mock-server)
- [TypeScript Best Practices](#typescript-best-practices)
- [Testing Quick Tips](#testing-quick-tips)
- [Happy Path and Sad Path](#happy-path-and-sad-path)
  - [Example: Login Flow](#example-login-flow)

---

## CSS and Layout Basics

### Display Types

- `block` elements start on a new line and fill the available width by default, so they are best for layout containers and fixed-width sections.
- `inline` elements stay in text flow and only take up as much space as their content, so they are best for labels, links, and small pieces of content.
- `inline-block` elements stay inline, but you can set width, height, and box spacing such as `margin-top` and `margin-bottom`.

### Overflow

- Use `overflow: hidden`, `overflow: auto`, or `overflow: scroll` when content exceeds the box.
- Pair overflow control with `block` or `inline-block` depending on whether the element should stay in flow or behave like a box.

### Positioning

- Keep `position: static` as the default; switch to `relative`, `absolute`, `fixed`, or `sticky` only when the layout needs it.
- Use `relative` on a parent when an absolutely positioned child should anchor to that container.
- Prefer Flexbox or Grid for the main layout; use positioning for overlays, badges, tooltips, popovers, and pinned UI.
- Remember that `absolute` removes an element from normal flow, so neighboring content will not reserve space for it.
- Use `sticky` for headers or sidebars that should remain in flow until they reach a scroll threshold.
- For visually hidden content, prefer the `.sr-only` pattern instead of moving elements off-screen with large negative positions.

- Example snippet: [Hiding content accessibly](./general-code-examples.md#hiding-content-accessibly)

### Centering

- Use Flexbox or Grid to center on both axes.
- Use `align-self` on a child when one item needs different cross-axis alignment than its siblings; it overrides the parent `align-items` for that item.
- Use `margin: 0 auto` for horizontal centering of a fixed-width block.
- Use `text-align: center` to center inline or text content.

### Spacing Shorthand

- Margin and padding follow clockwise order: top, right, bottom, left.
- One value applies to all sides, two values apply to vertical and horizontal, three values apply to top, horizontal, bottom, and four values apply to all sides.
- Logical properties such as `margin-block`, `margin-inline`, `padding-block`, and `padding-inline` are often easier to read.

### CSS Specificity

- Element selectors are weaker than class selectors, and class selectors are weaker than ID selectors.
- If two rules have the same specificity, the one declared later wins.
- Inline styles are stronger than normal stylesheet selectors.
- Use `!important` rarely and only when there is no cleaner option.

- Example snippet: [Block, inline, and inline-block](./general-code-examples.md#block-inline-and-inline-block)

Examples:

- `button` (1) < `.btn` (10) < `#submitBtn` (100)
- `.card .title` (20) beats `section h2` (2)
- `.btn.primary` (20) beats `.btn` (10)
- `section .ad-container #req-list ul li` (113) beats `body section .ad-container div .general-list li` (24)

## SOLID Principles

### Single Responsibility Principle (SRP)

> "There should never be more than one reason for a class to change." - Robert C. Martin

- Keep each component/hook focused on one responsibility.
- Split stateful/data logic from presentational rendering when complexity grows.
- Example snippets: [SRP Example](./general-code-examples.md#single-responsibility-srp)

### Open/Closed Principle (OCP)

> "Modules should be both open (for extension) and closed (for modification)." - Bertrand Meyer

- Prefer extension through composition (`children`, slots, wrappers) over modifying internals.
- Build reusable base components and extend them in feature components.
- Example snippets: [OCP Example](./general-code-examples.md#openclosed-ocp)

### Liskov Substitution Principle (LSP)

> "Let q(x) be a property provable about objects x of type T. Then q(y) should be true for objects y of type S where S is a subtype of T." - Barbara H. Liskov

- Specialized components should preserve base component behavior expectations.
- If a component wraps a native element, keep its core contract intact (props/events/accessibility).
- Example snippets: [LSP Example](./general-code-examples.md#liskov-substitution-lsp)

### Interface Segregation Principle (ISP)

> "Clients should not be forced to depend upon interfaces that they do not use." - Robert C. Martin

- Avoid oversized prop APIs; split interfaces by role and concern.
- Keep component contracts narrow, explicit, and easy to understand.
- Example snippets: [ISP Example](./general-code-examples.md#interface-segregation-isp)

### Dependency Inversion Principle (DIP)

> "A. High-level modules should not depend on low level modules. Both should depend on abstractions. B. Abstractions should not depend upon details. Details should depend upon abstractions." - Robert C. Martin

- Depend on abstractions (interfaces/contracts), not concrete implementations.
- Inject dependencies (API clients/services) to improve testability and reuse.
- Example snippets: [DIP Example](./general-code-examples.md#dependency-inversion-dip)

- Full SOLID code set: [SOLID Principles](./general-code-examples.md#solid-principles)

## Security Essentials

### XSS and Content Safety

- React escapes text by default when rendering values.
- Treat raw HTML rendering as high risk.
- Sanitize untrusted HTML before rendering.
- Validate and sanitize user-provided URLs.

### General Security Rules

- Never hardcode secrets in frontend code.
- Use HTTPS for all API calls.
- Validate user input client-side and server-side.
- Keep auth/session logic on secure backend boundaries.
- Avoid exposing sensitive data in logs or errors.

## When to Use Nullish Coalescing vs Logical OR

- Use `??` when the fallback should apply only to `null` or `undefined`.
- Use `||` when any falsy value should trigger the fallback.
- Prefer `??` for API values, counters, and form fields where `0`, `false`, or `""` can be valid.
- Prefer `||` when an empty or falsy value should be treated as missing.

```jsx
const count = 0;
const title = "";
const isAdmin = false;

count || 10; // 10
count ?? 10; // 0

title || "Untitled"; // "Untitled"
title ?? "Untitled"; // ""

isAdmin || true; // true
isAdmin ?? true; // false
```

Rule of thumb: if `0`, `false`, or an empty string are valid values, use `??` instead of `||`. If you want these values to trigger a fallback use `||`.

## Truthy and Falsy Values

- Truthy values behave like `true` in condition checks.
- Falsy values behave like `false` in condition checks.
- JavaScript falsy values are: `false`, `0`, `""`, `null`, `undefined`, and `NaN`.
- Most other values are truthy (including non-empty strings, non-zero numbers, arrays, and objects).
- Convert any value to a boolean with `!!value`. (converts the value to boolean and double-flips it)
- `Boolean(value)` does the same conversion and is often more readable.
- Avoid `new Boolean(false)`: it creates an object, and objects are truthy in conditions.

```jsx
if (value) {
  // Runs only when value is truthy
}

const a = !!value;
const b = Boolean(value); // same result as !!value
```

## Error Handling Essentials

### Error Boundary Strategy

- Use boundaries around risky UI zones, not only app root.
- Show fallback UIs that preserve key navigation paths.
- Log boundary-caught errors to monitoring systems.
- Keep boundary fallbacks actionable and user-friendly.

### What Boundaries Don't Catch

- Event handler errors.
- Async callback errors.
- Server-side rendering errors.
- Errors thrown inside the boundary itself.

### Practical Handling Rules

- Wrap async event logic with explicit try/catch.
- Normalize error objects before showing messages.
- Avoid exposing raw backend error details to users.

- Error handling snippets: [Code Examples](./general-code-examples.md#error-handling--rendering)

## Accessibility Quick Wins

- Use semantic elements before adding ARIA.
- Always associate labels with form controls.
- Ensure keyboard access for all interactive controls.
- Maintain visible focus indicators.
- Test color contrast in all themes/states.

### Text Color Contrast

- Ensure body text contrast is at least 4.5:1 against its background.
- Ensure large text (18px regular or 14px bold and larger) is at least 3:1.
- Ensure UI controls and focus indicators are at least 3:1 against adjacent colors.
- Avoid low-contrast placeholder text; users should still be able to read hint content.

### Alt Text for Images

- Provide meaningful `alt` text for informative images.
- Use `alt=""` for decorative images so screen readers skip them.
- Do not start alt text with "image of" or "photo of" unless that context matters.
- If nearby text already fully explains the image, keep alt text short and avoid repetition.

### ARIA Live Regions

- Use live regions for dynamic status messages that appear without focus change.
- Use `aria-live="polite"` for non-urgent updates (for example, "Saved" or filter results count).
- Use `aria-live="assertive"` only for urgent interruptions that must be announced immediately.
- Use `role="status"` for polite status messages and `role="alert"` for urgent errors.
- Keep announcements short and avoid updating the live region too frequently.
- Prefer one stable live region element and update its text instead of mounting many temporary nodes.

### Hiding Content Accessibly

- Hide content from everyone (visual + assistive tech) with `hidden`, `display: none`, or `visibility: hidden` when it is not relevant.
- Hide content visually but keep it available to screen readers using a visually-hidden utility class (for example `.sr-only`).
- Use `aria-hidden="true"` only for purely decorative or duplicate content.
- Never apply `aria-hidden="true"` to focusable/interactive elements.
- Do not rely on `opacity: 0` alone to hide important content; it may still be focusable/announced depending on markup.

### Better Links (Descriptive + Visible)

- Use descriptive link text that makes sense out of context.
- Avoid generic text such as "click here" or "read more" without context.
- Keep links visually identifiable (underline is recommended, at least on hover/focus).
- Distinguish links from regular text with more than color alone.

### Input Labels + Placeholder Usage

- Every input needs a visible label, even when the field seems obvious.
- Use placeholder text only for examples or format hints, not as the label.
- Keep labels concise and specific to reduce ambiguity in forms.
- Connect labels, help text, and errors using `htmlFor`, `id`, and `aria-describedby`.

### Fieldset and Legend for Grouped Controls

- Group related controls (radio groups, checkbox groups, address groups) with `fieldset`.
- Use `legend` as the accessible group title; it is announced by assistive tech.
- Keep legend text short and meaningful (for example, "Preferred contact method").

### Landmark Regions

- Structure pages with semantic landmarks: `header`, `nav`, `main`, `section`, `aside`, `footer`.
- Why: landmarks help screen reader and keyboard users jump quickly between major page regions. Also helps maintainability and clearer document structure.
- Keep one primary `main` landmark per page.
- Add labels for repeated landmarks when needed (`aria-label` for multiple `nav` elements).

### Use rem for Font Sizes

- Prefer `rem` for typography so text scales with user/browser settings.
- Base reference: `1rem = 16px` in default browser settings.
- Use fixed `px` values sparingly (for hairline borders or icon pixel alignment).

### Heading Structure

- Use one page-level `h1`.
- Keep heading levels sequential (`h1` -> `h2` -> `h3`) and avoid skipping levels.
- Use headings for document structure, not just visual styling.
- If only style is needed, use CSS classes on semantic elements instead of incorrect heading levels.

- Practical A11y examples: [Code Examples](./general-code-examples.md#accessibility--typescript)

## Critical Accessibility Principles

### Forms

- Label every input with visible, meaningful text.
- Expose validation state with `aria-invalid` and clear messages.
- Link errors/help text using `aria-describedby` when needed.
- Do not rely on color alone for error communication.

### Keyboard and Focus

- Preserve logical tab order.
- Avoid keyboard traps.
- Programmatically manage focus after major UI changes.
- Keep interactive hit areas comfortably large.
- Be careful with mouse-only events (`mouseover`, `mouseenter`, `mouseout`); they do not reliably work for keyboard, touch, or screen reader users.
- Prefer semantic controls and event patterns that support multiple input modes (`onFocus`/`onBlur`, `onClick`, keyboard handlers when needed).

### ARIA Usage

- Use ARIA only when native semantics are insufficient.
- Keep ARIA attributes synchronized with UI state.
- Avoid redundant or conflicting ARIA annotations.

## Form Handling Best Practices

- Keep submit state explicit (`idle/loading/success/error`).
- Disable submit during in-flight operations when appropriate.
- Validate inputs before request dispatch.
- Keep server-side validation as the source of truth.
- Use consistent error and success messaging patterns.

- Form snippets: [Code Examples](./general-code-examples.md#forms--api)

## API/Fetch Patterns

- Centralize request helpers for consistent headers and error handling.
- Handle cancellation for stale/unmounted requests.
- Treat non-2xx responses as explicit error states.
- Separate transport errors from domain/business errors.
- Keep retries/backoff policy explicit and bounded.

- API Examples: [Code Examples](./general-code-examples.md#apifetch-patterns)

## Mirage JS Mock Server

- Use Mirage JS to mock backend APIs during frontend development.
- Start Mirage only in development/test to avoid intercepting production traffic.
- Keep route contracts aligned with real API shapes (status codes + response schema).
- Seed realistic edge cases (empty lists, validation errors, latency).
- Centralize mock server setup in one file (for example `src/mocks/server.ts`).

- Mirage snippet: [Code Examples](./general-code-examples.md#mirage-js-setup)

## TypeScript Best Practices

- Type component props explicitly and narrowly.
- Prefer discriminated unions for variant-driven components.
- Avoid broad `any`; use `unknown` + refinement when needed.
- Keep shared domain types centralized and reusable.
- Type async results and error shapes consistently.

- TypeScript examples: [Code Examples](./general-code-examples.md#accessibility--typescript)

## Testing Quick Tips

- Test behavior and outcomes, not implementation details.
- Prefer user-centric queries and interactions.
- Cover loading, error, empty, and success states.
- Mock network boundaries, not internal React primitives.
- Keep tests deterministic and independent.

## Happy Path and Sad Path

- Always document and test the happy path first (expected input, expected flow, expected UI result).
- Include a short sad path list for common failures (validation errors, network failures, empty states, permission/auth issues).
- In overview docs, keep sad path coverage concise (top 3-5 scenarios), then link to deeper docs if needed.
- For tests, ensure each core feature has at least one happy path test and one sad path test.

### Example: Login Flow

- Happy path:
  - User enters valid email/password.
  - API returns 200 with user/session.
  - App stores session, redirects to dashboard, and shows authenticated UI.
- Sad path:
  - Wrong password: show inline error and keep user on login page.
  - Network timeout: show retry message and preserve typed input.
  - Server 500: show generic error (no internal details) and log error event.
