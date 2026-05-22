# React Overview

## Navigation Index

- [Quick Reference](#quick-reference)
  - [Golden Rules](#golden-rules)
  - [3 Phases of Rendering](#3-phases-of-rendering)
  - [SOLID Principles](#solid-principles)
  - [Common Pitfalls](#common-pitfalls)
  - [Performance Checklist](#performance-checklist)
  - [Security Essentials](#security-essentials)
  - [Quick Decision Trees](#quick-decision-trees)
  - [State Initialization](#state-initialization)
  - [When to Use Nullish Coalescing vs Logical OR](#when-to-use-nullish-coalescing-vs-logical-or)
  - [React 18+ Features & Patterns](#react-18-features--patterns)
  - [React 19 Features](#react-19-features)
  - [Error Handling Essentials](#error-handling-essentials)
  - [Conditional Rendering Patterns](#conditional-rendering-patterns)
  - [Fragment Best Practices](#fragment-best-practices)
  - [Accessibility Quick Wins](#accessibility-quick-wins)
  - [Critical Accessibility Principles](#critical-accessibility-principles)
  - [Form Handling Best Practices](#form-handling-best-practices)
  - [API/Fetch Patterns](#apifetch-patterns)
  - [Mirage JS Mock Server](#mirage-js-mock-server)
  - [React Router Essentials](#react-router-essentials)
  - [React.StrictMode (Development Tool)](#reactstrictmode-development-tool)
  - [Event Handling Gotchas](#event-handling-gotchas)
  - [Component Essentials](#component-essentials)
  - [List Keys Essentials](#list-keys-essentials)
  - [Props API Best Practices](#props-api-best-practices)
  - [Composition Patterns](#composition-patterns)
  - [Headless Patterns](#headless-patterns)
  - [Prop Drilling vs Composition vs Context](#prop-drilling-vs-composition-vs-context)
  - [Children Patterns](#children-patterns)
  - [Centering Elements](#centering-elements)
  - [Spacing Shorthand (Margin/Padding)](#spacing-shorthand-marginpadding)
  - [TypeScript Best Practices](#typescript-best-practices)
  - [Testing Quick Tips](#testing-quick-tips)
  - [Happy Path and Sad Path](#happy-path-and-sad-path)

---

# Quick Reference

## Golden Rules

### Hooks Rules (Non-negotiable)

- Call hooks only at component/custom-hook top level.
- Never call hooks in loops, conditions, or nested functions.
- Keep dependency arrays complete and accurate.
- Keep custom hook names prefixed with `use`.

### Component Essentials

- Keep components focused on one responsibility.
- Keep state close to where it is used.
- Prefer composition before deep prop drilling.
- Use stable IDs for list keys, not array indexes in dynamic lists.
- Treat state as immutable; always create new objects/arrays.

### Event Handlers

- Use clear naming (`handleX` for internal, `onX` for props).
- Avoid recreating handlers in large lists when it impacts performance.
- **Clean up listeners, timers, and async work on unmount.**
- Move complex logic out of JSX and into named functions.

## 3 Phases of Rendering

Every React update goes through three sequential phases:

### 1. Trigger

- React is told that a render is needed.
- Causes: initial render, a `setState` call, a context value change, or a parent re-render.
- React does not immediately update the DOM — it schedules work.

### 2. Render

- React calls your component functions to compute what the UI should look like.
- This is **pure** — no DOM writes, no side effects.
- React builds a new virtual DOM tree and diffs it against the previous one (reconciliation).
- A component only re-renders if its state, props, or context changed (or its parent did).

### 3. Commit

- React applies the minimal diff to the actual DOM.
- `useLayoutEffect` fires synchronously after DOM updates, before the browser paints.
- `useEffect` fires asynchronously after the browser has painted.
- Skipped entirely if the render phase found no changes.

## SOLID Principles

### Single Responsibility Principle (SRP)

> "There should never be more than one reason for a class to change." – Robert C. Martin

- Keep each component/hook focused on one responsibility.
- Split stateful/data logic from presentational rendering when complexity grows.
- Example snippets: [SRP Example](./code-examples.md#single-responsibility-srp)

### Open/Closed Principle (OCP)

> "Modules should be both open (for extension) and closed (for modification)." – Bertrand Meyer

- Prefer extension through composition (`children`, slots, wrappers) over modifying internals.
- Build reusable base components and extend them in feature components.
- Example snippets: [OCP Example](./code-examples.md#openclosed-ocp)

### Liskov Substitution Principle (LSP)

> "Let q(x) be a property provable about objects x of type T. Then q(y) should be true for objects y of type S where S is a subtype of T." – Barbara H. Liskov

- Specialized components should preserve base component behavior expectations.
- If a component wraps a native element, keep its core contract intact (props/events/accessibility).
- Example snippets: [LSP Example](./code-examples.md#liskov-substitution-lsp)

### Interface Segregation Principle (ISP)

> "Clients should not be forced to depend upon interfaces that they do not use." – Robert C. Martin

- Avoid oversized prop APIs; split interfaces by role and concern.
- Keep component contracts narrow, explicit, and easy to understand.
- Example snippets: [ISP Example](./code-examples.md#interface-segregation-isp)

### Dependency Inversion Principle (DIP)

> "A. High-level modules should not depend on low level modules. Both should depend on abstractions. B. Abstractions should not depend upon details. Details should depend upon abstractions." – Robert C. Martin

- Depend on abstractions (interfaces/contracts), not concrete implementations.
- Inject dependencies (API clients/services) to improve testability and reuse.
- Example snippets: [DIP Example](./code-examples.md#dependency-inversion-dip)

- Full SOLID code set: [SOLID Principles](./code-examples.md#solid-principles)

## Common Pitfalls

### State and Data Flow

- Direct state mutation causes stale UI and hard-to-debug bugs.
- Storing derived values in state creates sync issues.
- Duplicating source-of-truth state increases bugs.
- Using many unrelated state atoms can fragment logic.

### Effects and Dependencies

- Missing dependencies lead to stale closures.
- Effects used for pure calculations add unnecessary complexity.
- Side effects without cleanup cause memory leaks.
- Running network requests without cancellation risks race conditions.

### Lists and Rendering

- Index keys in dynamic lists can break item identity.
- Random keys force full remounts and lose local state.
- Heavy computations in render hurt responsiveness.
- Rendering large lists without virtualization impacts performance.

- Examples for these pitfalls: [Code Examples](./code-examples.md#state--effects)

## Performance Checklist

> 💡 Measure first. Don't prematurely optimize.

### When to Optimize

- Measure with React DevTools Profiler.
- Prioritize user-perceived lag (typing, scrolling, navigation).
- Optimize hot paths, not every component.
- Prefer architectural fixes before micro-optimizations.

### High-Impact Optimizations

- Memoize expensive derived data when profiling confirms need.
- Memoize components with `memo` when they re-render often with the same props.
- Stabilize callback references when passing to memoized children.
- Split large components and contexts by concern.
- Lazy-load heavy routes/components with `lazy` and `Suspense`.

### When to Use `memo`, `useMemo`, `useCallback`, and `lazy`

> 💡 First fix slow renders, then reduce unnecessary rerenders.

`memo` skips a re-render of a component, when its props are unchanged.

- Use `memo` for pure components that render frequently and usually receive the same props.
- Use `memo` when profiling shows parent re-renders are repeatedly cascading into unchanged child components.
- Do not use `memo` on every component by default; shallow prop comparison also has a cost.
- `memo` works best when prop identities are stable, often together with `useMemo` or `useCallback` where needed.

`useMemo` caches the result of a calculation between renders.

- Use `useMemo` when a derived value is expensive to recompute and the computation runs often enough to matter.
- Use `useMemo` when referential stability of a computed object/array matters for memoized children or effect dependencies.
- Do not use `useMemo` for cheap calculations or as a default optimization.

`useCallback` caches a function definition between renders.

- Use `useCallback` when passing handlers to memoized children and unstable function identity is causing avoidable re-renders.
- Use `useCallback` when a stable function reference is required by another hook or subscription setup.
- Do not use `useCallback` on every handler by default; it adds indirection without benefit unless identity matters.

`lazy` defers loading a component's code until its rendered for the first time.

- Use `lazy` for route-level code splitting and for heavy components that are not needed on initial load.
- Use `lazy` for infrequently used surfaces such as modals, admin panels, editors, or complex charts.
- Do not use `lazy` for tiny components above the fold where extra network round trips can hurt perceived performance.
- Do not forget to wrap the `lazy` loaded component in a `<Suspense>`, since its loaded on demand.

### Re-render Triggers to Watch

- Local state updates.
- Parent re-renders.
- Context value changes.
- Prop identity/value changes.

- Performance patterns: [Code Examples](./code-examples.md#performance)

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

## Quick Decision Trees

### State Management: Which Tool?

- Simple local state: `useState`.
- Complex local transitions: `useReducer`.
- Shared tree state: Context split by concern.
- Server state and caching: TanStack Query or SWR.
- App-wide complex state: Zustand or Redux Toolkit.
- URL-driven state: router state/query params.

### Which Hook?

- `useState`: simple local values.
- `useReducer`: multi-step/branching updates.
- `useEffect`: external side effects.
- `useMemo`: expensive derived computations.
- `useCallback`: stable function references.
- `useRef`: mutable instance value or DOM access. Example: [Code Examples](./code-examples.md#useref-for-dom-and-mutable-values)
- `useId`: accessible unique IDs.

## State Initialization

- Use lazy initialization for `useState` when initial computation is expensive.
- Use lazy initialization when reading one-time client values (for example, `localStorage`).
- Pass a function (`useState(() => initialValue)`) so initialization runs only on first render.
- Keep initializer functions pure and synchronous; do async bootstrap work in effects.

- Lazy initialization snippet: [Code Examples](./code-examples.md#lazy-initialization-with-usestate)

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

## React 18+ Features & Patterns

### Automatic Batching

- Multiple state updates in the same task are batched.
- Reduced re-render count improves responsiveness.

### Transitions

- Mark non-urgent updates with `startTransition`.
- Keep input updates urgent for better typing UX.
- Use `isPending` to communicate background work state.

### Suspense

- Use fallback UIs for lazy-loaded views.
- Keep fallback content minimal and meaningful.
- Place boundaries near slower feature surfaces.

## React 19 Features

### Forms and Async UX

- `useActionState` helps model action + pending + result flow.
- `useFormStatus` exposes submit/pending state from form context.
- `useOptimistic` supports optimistic UI while async work runs.

### Data and Component Patterns

- `use()` can simplify promise/context consumption in supported environments.
- Passing `ref` as a prop reduces `forwardRef` boilerplate.
- Ref cleanup callbacks improve lifecycle cleanup ergonomics.
- Improved metadata patterns help coordinate document head updates.

## Error Handling Essentials

### Error Boundary Strategy

- Use boundaries around risky UI zones, not only app root.
- Show fallback UIs that preserve key navigation paths.
- Log boundary-caught errors to monitoring systems.
- Keep boundary fallbacks actionable and user-friendly.

### What Boundaries Don’t Catch

- Event handler errors.
- Async callback errors.
- Server-side rendering errors.
- Errors thrown inside the boundary itself.

### Practical Handling Rules

- Wrap async event logic with explicit try/catch.
- Normalize error objects before showing messages.
- Avoid exposing raw backend error details to users.

- Error handling snippets: [Code Examples](./code-examples.md#error-handling--rendering)

## Conditional Rendering Patterns

- Prefer early returns for loading/error/empty branches.
- Keep branch logic shallow and readable.
- Extract complex conditions into named booleans.
- Return `null` intentionally when rendering nothing.

- Conditional rendering snippet: [Code Examples](./code-examples.md#early-return-conditional-rendering)

## Fragment Best Practices

- Use fragments to avoid unnecessary wrapper elements.
- Use keyed fragments when rendering multiple siblings in lists.
- Prefer semantic HTML over extra structural wrappers.

## Accessibility Quick Wins

- Use semantic elements before adding ARIA.
- Always associate labels with form controls.
- Ensure keyboard access for all interactive controls.
- Maintain visible focus indicators.
- Test color contrast in all themes/states.

- Practical A11y examples: [Code Examples](./code-examples.md#accessibility--typescript)

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

- Form snippets: [Code Examples](./code-examples.md#forms--api)

## API/Fetch Patterns

- Centralize request helpers for consistent headers and error handling.
- Handle cancellation for stale/unmounted requests.
- Treat non-2xx responses as explicit error states.
- Separate transport errors from domain/business errors.
- Keep retries/backoff policy explicit and bounded.

- API Examples: [Code Examples](./code-examples.md#apifetch-patterns)

## Mirage JS Mock Server

- Use Mirage JS to mock backend APIs during frontend development.
- Start Mirage only in development/test to avoid intercepting production traffic.
- Keep route contracts aligned with real API shapes (status codes + response schema).
- Seed realistic edge cases (empty lists, validation errors, latency).
- Centralize mock server setup in one file (for example `src/mocks/server.ts`).

- Mirage snippet: [Code Examples](./code-examples.md#mirage-js-setup)

## React Router Essentials

- Use nested routes + layout routes to share app chrome across pages.
- Render child routes with `Outlet` inside layout route components.
- Use `useOutletContext` to pass typed layout-level data to nested route components.
- Use `NavLink` for active-state navigation styling.
- With `NavLink`, use relative `to` values: `.` for current route and `..` for parent route.
- Read route params with `useParams` and URL query with `useSearchParams`.
- Use loaders/actions (Data Router) when route-level data and mutations fit better than ad-hoc effects.
- Prefer `useNavigate` for programmatic flows (post-submit redirects, auth guards).

- Router snippet: [Code Examples](./code-examples.md#react-router-basics)

## React.StrictMode (Development Tool)

- Use StrictMode in development to surface unsafe side effects.
- Expect additional development-only checks and re-invocations.
- Verify effects are idempotent and cleanup-safe.

## Event Handling Gotchas

- Do not call handlers during render; pass function references.
- Avoid expensive sync work directly in handlers.
- Debounce/throttle noisy events where needed.
- Keep handler side effects predictable and isolated.

## Component Essentials

- Keep components small and single-purpose; split when responsibilities diverge.
- Prefer function components and colocate feature-specific files.
- Keep component APIs explicit and discoverable; avoid hidden behavior.
- Use one component per file by default, except tiny tightly-coupled pieces.
- Avoid inline object/array literals in hot render paths when they cause re-renders.
- Provide sensible defaults for optional props in function signatures.

- Core component snippets: [Code Examples](./code-examples.md#components--composition)

## List Keys Essentials

- Every rendered list item needs a stable key.
- Use persistent IDs from data (`id`, UUID), not generated values during render.
- Avoid index keys for dynamic lists (reorder/add/remove) because identity breaks.
- Index keys are acceptable only for fully static lists.
- `key` is a React internals hint and is not available as a component prop.

- List key snippets: [Code Examples](./code-examples.md#components--composition)

## Props API Best Practices

- Prefer passing objects as props instead of many primitive values.
- Avoid a large amount of props (composition or similar can be used insteade)
- Be explicit with props; avoid broad `any` and catch-all prop dictionaries.
- Use prop spreading mainly to forward native DOM attributes intentionally.
- Group related props into objects (appearance/state/layout) once APIs start growing.
- Prefer discriminated unions for variant-driven components.
- Use `Readonly<Props>` for immutable component contracts where appropriate.

- Props snippets: [Code Examples](./code-examples.md#components--composition)

## Composition Patterns

- Use children for structural composition (wrappers/layout).
- Use data props for behavior/configuration.
- Combine children + data props for flexible component shells.
- Use compound components for tightly related UI with shared state
- For unstyled behavior-first variants, check headless pattern first
- Remember dot syntax can be used for compound components, when needed
- Split smart container (fetch/state/effects) from dumb presentational (UI-only) components.

- Composition snippets: [Code Examples](./code-examples.md#components--composition)

## Headless Patterns

- Use headless components when behavior/state should be reusable across different UIs.
- Keep styling concerns outside the logic layer (`className`, render props, or slots).
- Support controlled and uncontrolled modes for reusable inputs/toggles.

- Headless component snippets: [Code Examples](./code-examples.md#headless-component-pattern)

## Prop Drilling vs Composition vs Context

- Start with props for simple nearby parent-child data flow.
- If data passes through unrelated intermediates, prefer composition.
- Use Context when many distant components need the same cross-cutting state.
- Avoid Context for frequently changing local state unless measured and scoped.
- Rule of thumb: props first → composition second → Context when justified.

- Data-flow snippets: [Code Examples](./code-examples.md#prop-drilling-vs-composition-vs-context)

### Decision Tree

1. **Is the data needed by 1-2 adjacent components?** → Use **props**
2. **Does data need to skip multiple intermediate components?** → Try **composition** first
3. **Is data needed by many unrelated components?** → Use **Context** or **state management** (Redux, Zustand)
4. **Does data change frequently?** → Consider **state management library** instead of Context

## Children Patterns

- Use `children` for layout/wrapper composition.
- Prefer render props when children need dynamic data.
- Use compound components for tightly related UI pieces.
- Keep composition APIs consistent across component families.

- Children examples: [Code Examples](./code-examples.md#children-as-structure-data-as-behavior)

## Centering Elements

- To center on both axes, use a parent with Flexbox (`display: flex`) or Grid (`display: grid`).
- For horizontal-only centering of a fixed-width block, use `margin: 0 auto` (block-level element + set width/max-width).
- To center inline/text content, use `text-align: center` on the parent.

- Centering snippets: [Code Examples](./code-examples.md#centering-an-element)

## Spacing Shorthand (Margin/Padding)

- Shorthand order is clockwise: top, right, bottom, left.
- 1 value applies to all sides (for example `margin: 16px`).
- 2 values apply to vertical then horizontal (`top/bottom`, `left/right`).
- 3 values apply to `top`, `left/right`, `bottom`.
- 4 values apply to `top`, `right`, `bottom`, `left`.
- Axis shorthands are useful for readability: `margin-block`, `margin-inline`, `padding-block`, `padding-inline`.

- Spacing shorthand snippets: [Code Examples](./code-examples.md#marginpadding-shorthand)

## TypeScript Best Practices

- Type component props explicitly and narrowly.
- Prefer discriminated unions for variant-driven components.
- Avoid broad `any`; use `unknown` + refinement when needed.
- Keep shared domain types centralized and reusable.
- Type async results and error shapes consistently.

- TypeScript examples: [Code Examples](./code-examples.md#accessibility--typescript)

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
