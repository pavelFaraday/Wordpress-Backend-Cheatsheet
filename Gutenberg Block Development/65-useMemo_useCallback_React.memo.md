# `useMemo`, `useCallback`,`React.memo` in Gutenberg

- **`useMemo`**: remembers the _result of a calculation_ so you don’t redo it on every render.
  Think: “cache a value”.
- **`useCallback`**: remembers a _function reference_ so children don’t think the prop changed.
  Think: “stable handler”.
- **`React.memo`**: remembers a _component’s rendered output_ so it doesn’t re-render if its props didn’t meaningfully change.
  Think: “skip re-render when props are the same”.

In Gutenberg, every small change (typing, selecting block, opening sidebar) can re-render your block’s `edit` component. These three tools help **avoid unnecessary work**.

---

# Core concepts (with Gutenberg angles)

## 1) Referential equality

React compares props by reference. New arrays/objects/functions look “different” even if contents are the same.

- Bad:

  ```js
  <Child options={[1, 2, 3]} onClick={() => setAttributes({ open: true })} />
  ```

  New array and new function every render → `Child` keeps re-rendering.

- Good:

  ```js
  const options = useMemo(() => [1, 2, 3], []);
  const onClick = useCallback(
    () => setAttributes({ open: true }),
    [setAttributes]
  );

  <Child options={options} onClick={onClick} />;
  ```

## 2) Expensive derived values

If you compute something heavy (filter/sort/format/merge) from attributes or `useSelect` data, wrap it in `useMemo`.
**Only recomputes when its dependencies change.**

## 3) Stable props for memoized children

If a child uses `React.memo`, it only helps if you pass **stable** props (arrays/objects/handlers memoized with `useMemo`/`useCallback`).

## 4) `useSelect` and re-renders

- Select **exact primitives** (ids, booleans, counts).
- Avoid returning fresh objects directly from `useSelect`. If you must combine values, combine _outside_ with `useMemo`.

```js
const postId = useSelect(
  (select) => select("core/editor").getCurrentPostId(),
  []
);
// later
const combined = useMemo(
  () => ({ id: postId, label: `ID: ${postId}` }),
  [postId]
);
```

## 5) `React.memo` boundaries

Memo the **leaf** or mid-level presentational components that get many props (or lists).
Memoing the top `Edit` often has less impact because of editor state churn—focus on heavy subtrees.

---

# Practical use cases (Gutenberg block patterns)

## A) Memoize options for `<SelectControl>`

```js
import { SelectControl } from "@wordpress/components";
import { useMemo, useCallback } from "@wordpress/element";

export default function Edit({ attributes, setAttributes }) {
  const { color } = attributes;

  const colorOptions = useMemo(
    () => [
      { label: "Red", value: "red" },
      { label: "Green", value: "green" },
      { label: "Blue", value: "blue" },
    ],
    []
  );

  const onChangeColor = useCallback(
    (value) => {
      setAttributes({ color: value });
    },
    [setAttributes]
  );

  return (
    <SelectControl
      label="Color"
      value={color}
      options={colorOptions}
      onChange={onChangeColor}
    />
  );
}
```

**Why it helps:** `options` and `onChange` stay stable; child doesn’t re-render from prop identity churn.

## B) Heavy compute from attributes

```js
const sortedItems = useMemo(() => {
  // Assume attributes.items can be large
  return [...(attributes.items || [])].sort((a, b) =>
    a.title.localeCompare(b.title)
  );
}, [attributes.items]);
```

## C) Stable toolbar buttons & handlers

```js
import { BlockControls } from "@wordpress/block-editor";
import { ToolbarGroup, ToolbarButton } from "@wordpress/components";

const onToggle = useCallback(() => {
  setAttributes((prev) => ({ ...prev, enabled: !prev.enabled }));
}, [setAttributes]);

const toolbar = useMemo(
  () => (
    <BlockControls>
      <ToolbarGroup>
        <ToolbarButton icon="visibility" label="Toggle" onClick={onToggle} />
      </ToolbarGroup>
    </BlockControls>
  ),
  [onToggle]
);

return (
  <>
    {toolbar}
    {/* ...rest */}
  </>
);
```

**Why it helps:** the whole toolbar subtree stays stable unless the handler changes.

## D) Memoized list rows + `React.memo`

```js
import React, { useMemo, useCallback } from "@wordpress/element";

const Row = React.memo(function Row({ item, onRemove }) {
  // Pure presentational
  return (
    <div className="row">
      <span>{item.title}</span>
      <button onClick={() => onRemove(item.id)}>Remove</button>
    </div>
  );
});

export default function Edit({ attributes, setAttributes }) {
  const { items = [] } = attributes;

  const onRemove = useCallback(
    (id) => {
      setAttributes({ items: items.filter((i) => i.id !== id) });
    },
    [items, setAttributes]
  );

  const rows = useMemo(
    () =>
      items.map((item) => (
        <Row key={item.id} item={item} onRemove={onRemove} />
      )),
    [items, onRemove]
  );

  return <div>{rows}</div>;
}
```

**Key idea:** `Row` is `memo`-ized; `rows` only rebuild when `items`/`onRemove` change.

## E) Select only what you need (avoid fresh objects)

**Avoid:**

```js
const data = useSelect((select) => {
  return {
    post: select("core/editor").getCurrentPost(),
    user: select("core").getCurrentUser(),
  };
}, []);
// new object every render → consumers re-render
```

**Prefer:**

```js
const postTitle = useSelect(
  (select) => select("core/editor").getEditedPostAttribute("title"),
  []
);
const userId = useSelect((select) => select("core").getCurrentUser()?.id, []);
const data = useMemo(() => ({ postTitle, userId }), [postTitle, userId]);
```

## F) Batch attribute updates (reduce renders)

```js
// Good: single setAttributes
setAttributes({ title: value, subtitle: otherValue });

// If users type rapidly, debounce:
const onChangeTitle = useCallback(
  debounce((value) => {
    setAttributes({ title: value });
  }, 300),
  [setAttributes]
);
```

_(Use `lodash.debounce` available in WP packages or your own implementation.)_

---

# When _not_ to use them

- **Don’t memo everything.** Memoization has overhead. Use it for:

  - Expensive calculations
  - Props to `React.memo` children
  - Arrays/objects/handlers passed to deep trees or repeated lists

- **Don’t hide missing deps.** Always list real dependencies. If you skip them, you’ll get stale values (bugs that appear “randomly”).
- **`React.memo` doesn’t stop renders from parents changing context/state.** It only skips if the **props** are the same.

---

# Common pitfalls (and fixes)

1. **Anonymous props inside JSX**

```js
<PanelBody initialOpen={ true } /> // fine (primitive)
<Child style={{ padding: 8 }} />   // ❌ new object each render
```

Fix:

```js
const style = useMemo(() => ({ padding: 8 }), []);
<Child style={style} />;
```

2. **Recomputing selectors**
   If your `useSelect` returns a _new array/object_ from the store each time, it may still trigger re-renders. Prefer primitives or store selectors that already return memoized results; then assemble with `useMemo`.

3. **Stale closures**
   If you reference `items` inside a `useCallback` but don’t list it in deps, you’ll remove from an outdated list. Always include it (or use the functional `setAttributes(prev => ...)` pattern and derive from `prev`).

4. **Over-memoizing `Edit`**
   Memoize heavy children/components (lists, tables, tree views) rather than the top `Edit`.

---

# Mini performance checklist for Gutenberg

- [ ] In `useSelect`, select **primitives** or **stable IDs**.
- [ ] Wrap **derived arrays/objects** in `useMemo`.
- [ ] Wrap **handlers** in `useCallback`.
- [ ] Memoize heavy child components with `React.memo`.
- [ ] Batch `setAttributes`. Debounce rapid inputs.
- [ ] Avoid creating new objects/arrays/functions in JSX.
- [ ] Profile with React DevTools (“Highlight updates”) if unsure.

---

# Quick cheat-sheet

**`useMemo(factory, deps)`**

- Use for: expensive calculations, stable arrays/objects/JSX.
- Example:

  ```js
  const options = useMemo(
    () => items.map((i) => ({ label: i.name, value: i.id })),
    [items]
  );
  ```

**`useCallback(fn, deps)`**

- Use for: event handlers passed to children, callbacks used in `useEffect`/`useMemo`.
- Example:

  ```js
  const onSelect = useCallback(
    (id) => setAttributes({ selectedId: id }),
    [setAttributes]
  );
  ```

**`React.memo(Component, arePropsEqual?)`**

- Use for: presentational children that re-render a lot.
- Example:

  ```js
  const Tag = React.memo(function Tag({ label }) {
    return <span>{label}</span>;
  });
  ```

**Golden rules**

- Memo values you **pass down**.
- Include **all dependencies**.
- Memo where there’s **real churn or cost**, not everywhere.

---

# A realistic Gutenberg pattern (combined)

```js
import React, { useMemo, useCallback } from "@wordpress/element";
import { InspectorControls } from "@wordpress/block-editor";
import { PanelBody, SelectControl } from "@wordpress/components";

const ColorSwatch = React.memo(function ColorSwatch({
  value,
  onChange,
  options,
}) {
  // Pure UI; benefits from stable props
  return (
    <SelectControl
      label="Accent color"
      value={value}
      options={options}
      onChange={onChange}
    />
  );
});

export default function Edit({ attributes, setAttributes }) {
  const { accent, items = [] } = attributes;

  // Derived: turn items into select options (expensive if items is big)
  const itemOptions = useMemo(
    () =>
      items.map((i) => ({
        label: `${i.title} (${i.id})`,
        value: i.id,
      })),
    [items]
  );

  // Stable handlers
  const onChangeAccent = useCallback(
    (value) => setAttributes({ accent: value }),
    [setAttributes]
  );

  return (
    <>
      <InspectorControls>
        <PanelBody title="Design" initialOpen={false}>
          <ColorSwatch
            value={accent}
            onChange={onChangeAccent}
            options={itemOptions}
          />
        </PanelBody>
      </InspectorControls>

      {/* …rest of your block that reads accent/items… */}
    </>
  );
}
```
