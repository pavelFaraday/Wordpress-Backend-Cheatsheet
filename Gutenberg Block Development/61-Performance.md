# Performance & Best Practices in Gutenberg Block Development

Below is a compact but deep guide to building **fast, accessible, maintainable** blocks. It focuses on **render discipline**, **state modeling**, **component structure**, and **a11y**—plus a quick cheat-sheet and interview Q\&A.

---

## Why performance matters (in Gutenberg specifically)

- The editor is a **live React app** that re-renders as you type, select blocks, and change controls.
- Your block runs **in two places**: the canvas (visual block) and the sidebar (inspector controls). If your block’s code is written in a heavy or inefficient way, those bad patterns will run in both places — doubling the performance cost.

👉 Result: the editor feels slow — typing lags, selecting blocks feels sticky, and even publishing can take longer.

✅ Simple takeaway:
Write blocks to be light and efficient, because slow code affects the entire editing experience.

---

## Core mental models

### 1. **Attributes vs Local State**

- **Attributes** = saved data. Whatever goes here will be stored in the post or database and reloaded later.
  👉 Example: The text in a paragraph block.
- **Local state** = temporary memory inside React. It resets when you reload.
  👉 Example: Whether a settings panel is open/closed, or what you’ve typed but not saved yet.

✅ **Rule:** Only use attributes for things you actually want to save. Use local state for temporary UI stuff.

### 2. **Selective subscriptions**

- When you use `useSelect`, your block watches parts of the editor’s global state.
- If you subscribe to **too much**, your block re-renders too often → slow.
- Always pick the **smallest piece** of data you really need.
- If you return new objects/arrays, wrap them in `useMemo` so they don’t cause extra renders.

✅ **Rule:** Subscribe to less, not more.

### 3. **Re-render surfaces**

- Your block’s main `edit()` runs **a lot** while typing, selecting, or clicking. Keep it light.
- Heavy stuff (big previews, complex queries, lists) → move them into **child components**.
- Show them only when needed gated by conditions (like when the block is selected).

✅ **Rule:** Keep `edit()` simple; put heavy work behind conditions.

👉 **Simple takeaway:**

- Save only what matters (`attributes`).
- Watch less to render less (`useSelect`).
- Keep the main block lightweight (`edit()`), and push heavy UI into children.

---

## 🚀 How to Avoid Unnecessary Re-renders in Gutenberg Blocks

### 1) **Keep values and functions stable**

- Use **`useMemo`** for things you _calculate_ (like sorted lists, combined styles). This avoids recalculating them every render.
- Use **`useCallback`** for functions you pass down to child components (like button click handlers).
- Use **`React.memo`** to make child components re-render **only if their props really change**.
- If you pass arrays/objects as props (like `options` or `style`), wrap them in `useMemo` so React sees them as “the same” each render.

👉 **Rule:** Don’t create new objects/functions on every render if you can reuse them.

### 2) **Use `useSelect` carefully**

- Only grab the **exact piece of data you need** (like an ID, count, or single attribute).
- Don’t return a brand-new object each time inside `useSelect`. If you must combine things, wrap the result in `useMemo` to keep it stable.

👉 **Rule:** Select small, stable slices of data.

### 3) **Smarter attribute updates**

- Update multiple attributes at once in a **single `setAttributes` call**, instead of one by one.
- For text inputs: don’t save on **every keystroke**. Instead:

  - Keep text in **local state** while typing.
  - Save to attributes only when the field loses focus, or after a short delay (**debounce**).

👉 **Rule:** Save less often, in bigger batches.

### 4) **Don’t render heavy stuff unless needed**

- Show big/complex UI **only when the block is selected**.
- Delay expensive previews until the user clicks a toggle or button.
- Use placeholders or skeleton loaders instead of loading everything immediately.

👉 **Rule:** Render heavy things only when really needed.

✅ **Simple takeaway:**

- **Memoize** values/functions.
- **Select** only what you need.
- **Batch** attribute updates.
- **Gate** heavy UI behind conditions.

Would you like me to also create a **before vs after code snippet** (a “bad” version and an “optimized” version) so you can see how these tricks reduce re-renders in practice?

---

## ❗️🧩 Splitting Block Code into Smaller Components

### 1) **Divide by concern**

Break your block into parts:

- **CanvasView** → what shows in the editor (the block itself).
- **InspectorPanel** → the sidebar controls (settings).
- **Data/Preview** → any special logic or heavy preview.

👉 This makes each piece easier to read, test, and optimize.

### 2) **Keep children pure**

- Write child components so they **only depend on props** (data passed in).
- No hidden state inside if not needed.
- Wrap with **`React.memo`** so they don’t re-render unless props really change.

👉 Think of children as “dumb” components that just display what you give them.

### 3) **Pass minimal props**

- Don’t pass huge objects if the child only needs one value.
  👉 Example: instead of sending the whole `post` object, just send `post.id`.

### 4) **Isolate heavy logic**

- Put big tasks (like previews or lists) into **their own component**.
- Inside that child, use its own `useSelect` for data.
- Memoize locally so it doesn’t slow down the main block.

✅ **Simple takeaway:**
Keep your main block light and split big tasks into smaller, focused components. This makes your block **faster, easier to maintain, and easier to test**.

**Pattern:**

- `BlockEdit` (light wrapper)
  → `CanvasView` (visual block UI)
  → `InspectorPanel` (sidebar controls)
  → `HeavyPreview` (conditionally mounted, memoized)

---



## 🌐 Data Access, Fetching, and SSR (Server-Side Rendering)

### 1) **Use WordPress stores for data in the editor**

- WordPress already gives you data stores (`@wordpress/data`, `@wordpress/core-data`).
- These let you grab things like posts, users, taxonomies directly in React.
- Always select **only the fields you need**, not the whole object.

👉 Example: Instead of fetching all post data, just get the post title and ID.

### 2) **Cache heavy lookups**

- If you’re calling a function that builds big lists (like posts by category), **cache results** by using memoization.
- Key the cache by something stable (like post ID).
- This way, React doesn’t redo heavy work every time it re-renders.

### 3) **Server-rendered HTML (SSR)**

- For the **front end**, use PHP (`render_callback`) to generate the final markup — it’s safer and faster.
- In the **editor**, don’t render the full heavy HTML. Instead, show a **lighter preview**.

  - Sometimes use **SSR in the editor** too, but debounce it so it doesn’t run constantly while typing.

### 4) **Throttle and debounce expensive tasks**

- Remote API calls or complex computations shouldn’t fire on _every keystroke_.
- Use **debouncing** (wait until the user stops typing) or **throttling** (limit how often it runs).
- WordPress has `useDebounce`, or you can write a small custom hook.

👉 Example: When searching posts, wait 300ms after typing before sending the query.

✅ **Simple takeaway:**

- Use WordPress stores for editor data.
- Cache heavy lookups.
- Render final HTML on the server, light preview in the editor.
- Debounce/throttle expensive calls.

---

## ♿ Accessibility Basics (for Gutenberg Blocks)

### 1) **Focus management**

- When someone selects your block, move the cursor (focus) to the **main control** only if it helps.
- Don’t “trap” the user inside your block — they should be able to press **ESC** to exit popups or move focus away.

👉 Rule: Help users, don’t lock them in.

### 2) **Keyboard support**

- Everything should work with just a **keyboard** (no mouse needed).
- Show a clear **focus outline** (so users can see where they are).

👉 Rule: Users must be able to reach and use every button with Tab + Enter/Space.

### 3) **Labels & announcements**

- If a button only shows an icon (like a gear ⚙️), give it an **`aria-label`** so screen readers know what it does.
- Link inputs with labels.
- If something changes automatically (like a preview updating), let screen readers know.

👉 Rule: Don’t leave screen reader users guessing.

### 4) **Roles and structure**

- Use **real HTML tags** first: `<button>`, `<h2>`, `<ul>`.
- Only add roles (`role="dialog"`) when you really need them (like in a popup/modal).
- If it’s a dialog, connect it with a title (`aria-labelledby`).

👉 Rule: Use semantic HTML before fancy ARIA.

### 5) **Color & contrast**

- Always use colors that are easy to read (good contrast).
- Don’t hardcode colors — respect theme settings so users get accessible combinations.

👉 Rule: Text must stand out from the background.

### 6) **Avoid contentEditable traps**

- Don’t use raw `contentEditable` divs for text editing.
- Use Gutenberg’s **`RichText`** instead — it already has accessibility built in.

👉 Rule: Use Gutenberg tools, they come with a11y for free.

✅ **Simple takeaway:**
Make sure your block works for _everyone_ — keyboard users, screen readers, low-vision users. Use standard Gutenberg components and good HTML, and you’ll cover most accessibility needs.

---

## Additional micro-optimizations

- **Icons:** use lightweight SVGs (from `@wordpress/icons`) rather than large images.
- **Styles:** co-locate minimal CSS; prefer logical properties; avoid deeply nested selectors with high specificity.
- **Assets:** lazy-load only when needed (e.g., heavy preview logic behind a toggle).
- **Logging:** remove dev `console.*` in production builds.

---

## Quick Cheat-Sheet (copy & pin)

**Render discipline**

- Keep `edit()` light; push heavy work to child components.
- Guard heavy UI behind `isSelected` or explicit toggles.

**Memoization**

- `useMemo` for derived data; `useCallback` for event handlers passed to memoized children; `React.memo` children.

**State modeling**

- Persist only what you must (attributes/meta).
- Keep transient UI state in React (not attributes).
- Batch `setAttributes`; debounce text changes.

**Selectors**

- Select minimal slices with `useSelect`; avoid returning new object shapes; memoize derived props.

**A11y**

- Manage focus on selection; label icon buttons; ensure keyboard paths & semantics; maintain color contrast.

**Effects**

- Prefer `useEffect`; reserve `useLayoutEffect` for measurement.
- Throttle/ debounce expensive work; clean up observers.

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

---

## Main Rules

✅ **Rule:** Write blocks to be light and efficient, because slow code affects the entire editing experience.
✅ **Rule:** Only use attributes for things you actually want to save. Use local state for temporary UI stuff.
✅ **Rule:** Subscribe to less, not more.
✅ **Rule:** Keep `edit()` simple; put heavy work behind conditions.
✅ **Rule:** Use WordPress stores for editor data.
✅ **Rule:** Cache heavy lookups.
✅ **Rule:** Render final HTML on the server, light preview in the editor.
✅ **Rule:** Debounce/throttle expensive calls.
