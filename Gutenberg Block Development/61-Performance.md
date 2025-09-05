# Performance & Best Practices in Gutenberg Block Development (WP 6+)

Below is a compact but deep guide to building **fast, accessible, maintainable** blocks. It focuses on **render discipline**, **state modeling**, **component structure**, and **a11y**—plus a quick cheat-sheet and interview Q\&A.

---

## Why performance matters (in Gutenberg specifically)

* The editor is a **live React app** that re-renders as you type, select blocks, and change controls.
* Your block runs **in two places**: the canvas (visual block) and the sidebar (inspector controls). Poor patterns multiply cost.
* Slow blocks degrade typing latency, selection lag, and publishing flow.

---

## Core mental models

1. **Attributes vs Local State**

* **Attributes** are persisted to content (or meta) and trigger global editor updates.
* **Local React state/refs** are **ephemeral** and cheap; use them for transient UI (e.g., input drafts, open/closed toggles).
  **Rule:** Persist only what you intend to save or need for server output; keep UI-only state local.

2. **Selective subscriptions**

* `useSelect` re-runs when **any selected store value changes**. Select the **smallest slice** you need. Avoid returning new object/array **shapes** every time (wrap with `useMemo`).

3. **Re-render surfaces**

* The **block’s `edit()`** component is re-rendered often; keep it lean.
* Heavy tools (previews, complex lists, queries) belong in **child components** gated by conditions (e.g., only when `isSelected`).

---

## Avoiding unnecessary re-renders

### 1) Stable values & handlers

* **`useMemo`** for **derived values** (sorted lists, computed styles, merged props) that are expensive or frequently created.
* **`useCallback`** for **props you pass to memoized children** (buttons, inputs). If a child uses `React.memo`, unstable inline functions/objects will still cause re-renders.
* **`React.memo`** for **pure child components** that re-render only when their props change meaningfully.
* Memoize **arrays/objects** you pass as props (e.g., `options`, `style`) so their identity is stable.

### 2) Narrow `useSelect` usage

* Select **exact values** (IDs, counts, a specific attribute) rather than whole entities.
* Avoid returning anonymous **new objects** (e.g., `{ post, author }`) directly from `useSelect`; construct them with `useMemo` in the component.

### 3) Attribute updates (write discipline)

* **Batch updates** with a single `setAttributes` call for related fields.
* **Debounce** high-frequency updates (e.g., on text input) so you don’t flood the editor; keep draft text in local state and persist on blur or after a short delay.

### 4) Render gating

* Wrap heavy UI in conditions:

  * Only show complex controls **when the block is selected**.
  * Defer costly previews until a toggle is on, or content is present.
* Replace expensive areas with **skeletons** or “Load preview” buttons during editing.

---

## Splitting code into smaller components

* **Divide by concern:** canvas rendering, inspector controls, data fetching, preview.
* **Keep children pure:** stateless, prop-driven, `React.memo`-wrapped.
* **Pass minimal props:** avoid passing entire objects; pass only what’s needed (ids, flags).
* **Isolate heavy pieces:** complex preview/list → own component with local memoization and its **own** `useSelect` slice.

**Pattern:**

* `BlockEdit` (light wrapper)
  → `CanvasView` (visual block UI)
  → `InspectorPanel` (sidebar controls)
  → `HeavyPreview` (conditionally mounted, memoized)

---

## Data access, fetching, and SSR

* Prefer **client-side data** via WordPress stores (`@wordpress/data`, `@wordpress/core-data`) for editor interactivity; select only what you need.
* **Cache** expensive selectors using normalized keys (e.g., by ID) and memoized maps.
* For server-templated HTML, render on the front end with PHP; in the editor, use a **lightweight preview** (possibly SSR, debounced).
* **Debounce/throttle** remote calls and expensive computations; consider `useDebounce` (or a tiny custom hook).

---

## DOM, layout, and effects

* Avoid layout thrashing: read DOM once, then write; prefer CSS for effects over JS calculations.
* Only use **`useLayoutEffect`** when you must measure and synchronously reflow; otherwise use **`useEffect`** (cheaper).
* Use **`ResizeObserver`** sparingly; observe the **smallest** element possible and disconnect on unmount.

---

## Accessibility (focus handling & ARIA)

* **Focus management:** When a block becomes **selected**, move focus to the primary interactive control *only if it helps*. Don’t trap focus; allow ESC to exit popovers.
* **Keyboard support:** Ensure all actions are reachable by keyboard. Provide clear focus styles.
* **Labels & announcements:** Icon-only buttons need **`aria-label`**; associate labels with inputs; announce changes to screen readers if content updates dynamically.
* **Roles and structure:** Prefer **semantic HTML** (headings, lists, buttons) before adding `role`. Use `role="dialog"` + `aria-labelledby` for modals/popovers when appropriate.
* **Color/contrast:** Respect theme tokens; avoid hard-coded colors that violate contrast.
* **Avoid `contentEditable` traps:** Use Gutenberg’s `RichText` and standard components; they carry a11y behaviors.

---

## Additional micro-optimizations

* **Icons:** use lightweight SVGs (from `@wordpress/icons`) rather than large images.
* **Styles:** co-locate minimal CSS; prefer logical properties; avoid deeply nested selectors with high specificity.
* **Assets:** lazy-load only when needed (e.g., heavy preview logic behind a toggle).
* **Logging:** remove dev `console.*` in production builds.

---

## Quick Cheat-Sheet (copy & pin)

**Render discipline**

* Keep `edit()` light; push heavy work to child components.
* Guard heavy UI behind `isSelected` or explicit toggles.

**Memoization**

* `useMemo` for derived data; `useCallback` for event handlers passed to memoized children; `React.memo` children.

**State modeling**

* Persist only what you must (attributes/meta).
* Keep transient UI state in React (not attributes).
* Batch `setAttributes`; debounce text changes.

**Selectors**

* Select minimal slices with `useSelect`; avoid returning new object shapes; memoize derived props.

**A11y**

* Manage focus on selection; label icon buttons; ensure keyboard paths & semantics; maintain color contrast.

**Effects**

* Prefer `useEffect`; reserve `useLayoutEffect` for measurement.
* Throttle/ debounce expensive work; clean up observers.

---

## Interview-Style Talking Points (with strong model answers)

**Q1. What’s the #1 performance win in Gutenberg blocks?**
**A.** Reduce avoidable re-renders. Keep `edit()` lean, split heavy logic into memoized children, and select minimal state with `useSelect`. Memoize derived values (`useMemo`) and handlers (`useCallback`) so memoized children don’t re-render unnecessarily.

**Q2. How do `useMemo` and `useCallback` actually help here?**
**A.** They provide **referential stability**. Expensive computations aren’t re-run unless deps change, and child components that depend on stable props (functions/objects) won’t re-render when nothing meaningful changed.

**Q3. Attribute updates are making my block feel laggy—what do I do?**
**A.** Don’t persist on every keystroke. Keep the input value in local state, **debounce** writes to `setAttributes`, or save on blur. Batch related updates into one call.

**Q4. How should I use `useSelect` without causing churn?**
**A.** Select the **smallest** needed values (e.g., IDs, booleans, counts), not big objects. Avoid constructing new objects/arrays in the selector; if needed, memoize them in the component.

**Q5. When do you split components?**
**A.** When a piece is **complex or independent** (previews, lists, inspectors). Wrap in `React.memo` and pass only minimal, stable props. That confines re-renders to the parts that change.

**Q6. What are common a11y mistakes in custom blocks—and fixes?**
**A.** Missing labels on icon buttons (add `aria-label`), unclear focus order (move focus intentionally on selection/open), non-semantic tags (use `<button>`, `<ul>/<li>`, headings). For dialogs/popovers, set proper roles and labelled-by relationships; keep everything keyboard-accessible.

**Q7. How do you handle dynamic/expensive previews?**
**A.** Make them **opt-in** (toggle or only when selected), show a **skeleton** while loading, **debounce** data requests, and cache where possible. If the front end is PHP-rendered, consider a **light SSR** preview only while editing.

**Q8. What’s your strategy for measuring performance regressions?**
**A.** Use the React Profiler, test typing latency in long posts, and add small perf marks around heavy code paths. Monitor re-render counts during key interactions (typing, selection, sidebar toggles).

**Q9. Any tips for working well with themes?**
**A.** Respect `theme.json` tokens (palette, spacing, typography). Don’t hardcode colors; inherit and expose only the supports you need to keep UI light and on-brand.

**Q10. One pitfall you avoid now that you didn’t before?**
**A.** Passing fresh inline objects/functions to memoized children—looked harmless, but it invalidated memoization. Stabilizing props with `useMemo`/`useCallback` cut re-renders dramatically.

