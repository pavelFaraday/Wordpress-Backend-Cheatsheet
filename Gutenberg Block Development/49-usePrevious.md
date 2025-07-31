# `usePrevious` hook in Gutenberg

The `usePrevious` hook from `@wordpress/compose` is a highly useful utility in **WordPress Gutenberg block development**, especially when you're building dynamic blocks that depend on **comparing current and previous states or props**.

## 🧠 What is `usePrevious`?

The `usePrevious` hook is a custom hook provided by the `@wordpress/compose` package. It allows you to **retain the previous value** of a variable (usually a prop or state) across renders.

```js
import { usePrevious } from "@wordpress/compose";
```

It behaves similarly to the common React pattern:

```js
const prevValue = useRef();
useEffect(() => {
  prevValue.current = value;
});
```

But `usePrevious` wraps that logic for you in a clean, reusable hook.

---

### Example:

```js
import { usePrevious } from "@wordpress/compose";

const prevURL = usePrevious(url);

useEffect(() => {
  if (url && !prevURL) {
    titleRef.current?.focus();
  }
}, [url, prevURL]);
```

This useEffect hook runs every time either url or prevURL changes.
The usePrevious hook stores the previous value of the url state/prop between renders.
So, if url changes across renders, prevURL will hold its previous value before the change.

| Line                        | What it Does                              |
| --------------------------- | ----------------------------------------- |
| `usePrevious(url)`          | Saves previous `url` value for comparison |
| `useEffect(...)`            | Reacts when the `url` changes             |
| `if (url && !prevURL)`      | Detects the _first time_ a URL is added   |
| `titleRef.current?.focus()` | Focuses the title input for UX            |

---

## 🔧 Core Concept

The hook stores the **previous render's value** and gives you access to it during the **current render**.

### Signature:

```js
const prevValue = usePrevious(value);
```

- `value`: Any variable (usually state or prop).
- `prevValue`: The value of `value` from the previous render.

---

## 🎯 Purpose in Gutenberg Block Development

In Gutenberg, you often deal with block `attributes`, user selections, editor state changes, etc. If you want to perform some logic **only when a specific prop changes** or compare previous and current attribute values, `usePrevious` is the way to go.

---

## ✅ Common Use Cases in Gutenberg

### 1. **Detecting Prop/Attribute Changes**

Let’s say you want to trigger a function only when a block attribute changes _from_ something specific _to_ something else:

```js
const { myAttr } = attributes;
const prevAttr = usePrevious(myAttr);

useEffect(() => {
  if (prevAttr !== undefined && myAttr !== prevAttr) {
    console.log(`Attribute changed from ${prevAttr} to ${myAttr}`);
    // Perform logic here
  }
}, [myAttr]);
```

> This is great for syncing internal state when external data (like `attributes`) changes.

---

### 2. **Debounced Actions (Preventing Over-Reactions)**

Avoid triggering an update when the value hasn't really changed:

```js
const prevValue = usePrevious(inputValue);

useEffect(() => {
  if (prevValue !== inputValue) {
    // Only run this effect when inputValue truly changed
    performLivePreview(inputValue);
  }
}, [inputValue]);
```

---

### 3. **Preventing Redundant Updates**

Sometimes, an external function (like updating a third-party API or DOM element) shouldn't fire unless the value actually changed.

```js
const prevId = usePrevious(attributes.imageId);

useEffect(() => {
  if (prevId !== attributes.imageId) {
    updateImagePreview(attributes.imageId);
  }
}, [attributes.imageId]);
```

---

### 4. **Custom Animation or UI Transition Logic**

When implementing custom block UIs that react to value changes:

```js
const prevIsSelected = usePrevious(isSelected);

useEffect(() => {
  if (!prevIsSelected && isSelected) {
    // Block was just selected – play animation or show toolbar
  }
}, [isSelected]);
```

---

## 🧪 Practical Example: Image Upload in Block

Let’s say your Gutenberg block allows users to upload an image and shows a preview. You want to display a spinner **only when a new image is uploading**.

```js
import { useEffect, useState } from "@wordpress/element";
import { usePrevious } from "@wordpress/compose";
import { MediaUpload } from "@wordpress/block-editor";

export default function Edit({ attributes, setAttributes }) {
  const { url } = attributes;
  const [isLoading, setIsLoading] = useState(false);
  const prevUrl = usePrevious(url);

  useEffect(() => {
    if (prevUrl !== url && isBlobURL(url)) {
      setIsLoading(true);
    } else {
      setIsLoading(false);
    }
  }, [url]);

  return (
    <div>
      <MediaUpload
        onSelect={(image) => setAttributes({ url: image.url })}
        allowedTypes={["image"]}
        render={({ open }) => <button onClick={open}>Upload</button>}
      />
      {isLoading && <Spinner />}
      {!isLoading && url && <img src={url} alt="" />}
    </div>
  );
}
```

---

## 📦 How to Add It (if not already available)

If you're using `@wordpress/scripts`, `usePrevious` is already included via `@wordpress/compose`.

But if not:

```bash
npm install @wordpress/compose
```

---

## 🔍 Source Code Insight

Here’s a simplified version of what `usePrevious` does internally:

```js
import { useRef, useEffect } from "@wordpress/element";

export function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  });

  return ref.current;
}
```

> It's a simple wrapper around `useRef()` that captures the value _after_ each render.

---

## 🧠 Pro Tips

- Avoid comparing deep objects (like `attributes`) directly — use individual keys or deep comparison if needed.
- Combine `usePrevious` with `useEffect` to create fine-grained control over side effects in blocks.
- Extremely useful in **multi-step forms, toggles, previews, or undo-like behaviors** in custom Gutenberg blocks.

---

## 📌 Summary

| Feature       | Details                                                  |
| ------------- | -------------------------------------------------------- |
| Hook Name     | `usePrevious`                                            |
| Package       | `@wordpress/compose`                                     |
| Purpose       | Track previous state/prop value                          |
| Key Use Cases | Attribute change detection, UI transition, form tracking |
| Benefit       | Clean, readable state comparison                         |

---

Would you like a ready-to-use boilerplate block code using `usePrevious` for your project?
