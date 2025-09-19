# `useMemo`, `useCallback`,`React.memo` in Gutenberg

- **`useMemo`**: remembers the _result of a calculation_ so you don’t redo it on every render.
  Think: “cache a value”.
- **`useCallback`**: remembers a _function reference_ so children don’t think the prop changed.
  Think: “stable handler”.
- **`React.memo`**: remembers a _component’s rendered output_ so it doesn’t re-render if its props didn’t meaningfully change.
  Think: “skip re-render when props are the same”.

In Gutenberg, every small change (typing, selecting block, opening sidebar) can re-render your block’s `edit` component. These three tools help **avoid unnecessary work**.

✅ **One-liner summary for interview**

- `useMemo`: cache a **value** → skip recalculations.
- `useCallback`: cache a **function** → stable props.
- `React.memo`: cache a **component’s output** → skip useless re-renders.

✅ **Interview one-liner**
“In Gutenberg, since the editor triggers many renders, I use `useMemo` for expensive values, `useCallback` for stable handlers, and `React.memo` for children. Together they keep blocks fast and prevent unnecessary re-renders.”

🧩 **How they combine in Gutenberg**

- Use **`useMemo`** to cache heavy values (sorted posts, generated styles).
- Use **`useCallback`** to keep handlers stable when passing to children (`TextControl`, `MediaUpload`, custom components).
- Wrap **child components** with **`React.memo`** so they only re-render if their props change.

This pattern makes blocks smoother in the editor — especially important since **Gutenberg re-renders your block often** (typing, selecting, opening sidebars).

---

## 🔑 React Performance Helpers in Simple Words

### 1) `useMemo` — _cache a value_

- **What it does:** Remembers the result of a calculation so React doesn’t redo it on every render.
- **Think:** _“Don’t waste time recalculating if inputs didn’t change.”_
- **Example:** Sorting a big list of products. Without `useMemo`, it re-sorts on every keystroke, even if the list hasn’t changed. With `useMemo`, React remembers the sorted list until the list itself changes.

### 2) `useCallback` — _stable function reference_

- **What it does:** Remembers a function so its reference doesn’t change on every render.
- **Think:** _“Keep my handler the same unless inputs change.”_
- **Example:** A `Button` child gets an `onClick` function from the parent. If the parent creates a new function every render, the child thinks “my props changed” and re-renders. With `useCallback`, the function reference stays stable, so the child won’t re-render unnecessarily.

### 3) `React.memo` — _skip re-render if props didn’t change_

- **What it does:** Wraps a component so React reuses the last render if the props are the same.
- **Think:** _“Don’t redraw this child if nothing about it changed.”_
- **Example:** A `ProductCard` that only depends on its product props. If the parent re-renders but the product didn’t change, React.memo prevents extra work.

## 🧩 How they work together

- `React.memo` checks props → if same, **skip render**.
- `useCallback` makes sure function props _look the same_ between renders.
- `useMemo` makes sure heavy values/objects _look the same_ between renders.

Together they prevent unnecessary re-renders and heavy recalculations.

## 🚦 Interview-Style Talking Points

**Q1. Why do we need these hooks in React?**
👉 Because React re-renders components a lot. That’s fine for small UIs, but wasteful if you recalc heavy data or recreate props every time. These tools cut down on useless work.

**Q2. Can you explain `useMemo` in plain English?**
👉 It’s like a **calculator with memory**. If I already calculated 2+2=4 and the numbers haven’t changed, I don’t need to recalc—it just gives me 4 again. That saves time for big calculations.

**Q3. What’s the difference between `useMemo` and `useCallback`?**
👉 `useMemo` caches a **value**.
👉 `useCallback` caches a **function**.
Both avoid creating “new things” on every render, which helps React.memo work properly.

**Q4. How does `React.memo` actually help performance?**
👉 It’s like saying: _“If the inputs didn’t change, just reuse the picture we already drew instead of redrawing it.”_ Props stay the same → no re-render → faster app.

**Q5. Any caveats?**
👉 Yes:

- Don’t wrap **everything** in useMemo/useCallback—it adds complexity.
- Only use them for **expensive calculations** or when child components re-render too often.
- Overusing them can actually make code harder to read and slower (because React still has to compare dependencies).

---

# ⚡ Practical Gutenberg Examples

### 1) `useMemo` — cache a value

**Scenario:** You have a block that lists posts. Sorting or filtering the posts is expensive. Without `useMemo`, it re-sorts on every keystroke in the block editor.

```js
import { useMemo } from "@wordpress/element";

export default function Edit({ attributes }) {
  const { posts } = attributes;

  // Sorting posts alphabetically
  const sortedPosts = useMemo(() => {
    console.log("Sorting…");
    return [...posts].sort((a, b) => a.title.localeCompare(b.title));
  }, [posts]);

  return (
    <ul>
      {sortedPosts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

👉 Now the list only re-sorts if `posts` actually change — not every time you select or deselect the block.

---

### 2) `useCallback` — stable function reference

**Scenario:** You pass an `onChange` handler to a child input. Without `useCallback`, the function reference changes every render, so the child thinks “props changed” → re-renders.

```js
import { useCallback } from "@wordpress/element";
import { TextControl } from "@wordpress/components";

export default function Edit({ attributes, setAttributes }) {
  const { title } = attributes;

  // Stable handler
  const onChangeTitle = useCallback(
    (newValue) => {
      setAttributes({ title: newValue });
    },
    [setAttributes]
  );

  return (
    <TextControl
      label="Title"
      value={title}
      onChange={onChangeTitle} // stable reference
    />
  );
}
```

👉 Prevents unnecessary re-renders of `TextControl` and keeps typing smooth.

---

### 3) `React.memo` — skip re-render if props didn’t change

**Scenario:** A block renders a list of child cards. Each card doesn’t need to re-render unless its props change.

```js
import { memo } from "@wordpress/element";

const Card = memo(function Card({ title }) {
  console.log("Rendering card:", title);
  return <div className="my-card">{title}</div>;
});

export default function Edit({ attributes }) {
  const { items } = attributes;

  return (
    <div>
      {items.map((item, i) => (
        <Card key={i} title={item} />
      ))}
    </div>
  );
}
```

👉 If only one `item` changes, **only that Card re-renders**. The others are skipped.
