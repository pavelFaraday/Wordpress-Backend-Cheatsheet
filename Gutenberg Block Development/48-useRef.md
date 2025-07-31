# `useRef` in Gutenberg

`useRef` is a powerful React Hook, and in the context of **WordPress Gutenberg block development**, it plays a crucial role when you need to:

- Directly **access DOM elements** in your custom blocks.
- **Persist values** across renders without triggering a re-render.
- Control **focus**, **measurements**, or work with **third-party libraries** that require a DOM node.

Below is a **deep dive** into how and why you'd use `useRef` in **Gutenberg block development**.

---

## 🚀 What is `useRef`?

`useRef` is a **React hook** that returns a **mutable object**. This object contains a `.current` property, which you can assign to a DOM element (or store any value), and **React will not change it between renders**.

```js
const myRef = useRef();
```

The object returned stays the same **between renders**, making it ideal for:

- Referencing DOM elements
- Storing mutable data that doesn’t trigger re-renders

---

## 🧠 Core Concepts Behind `useRef`

1. **DOM Manipulation**: Get direct access to DOM elements without triggering re-renders.
2. **Persisting Data Without Re-render**: Unlike `useState`, changes to `ref.current` don’t cause re-renders.
3. **Avoiding Re-initialization**: Use `useRef` to hold values you want to keep constant between renders but don’t want to recalculate or re-initialize.

---

## 📦 How `useRef` Works in Gutenberg

Gutenberg blocks are built with **React under the hood**, so `useRef` works just like it does in any other React environment. It comes in handy when:

- You’re creating **editable elements** or **custom input fields**.
- You want to **focus** an input when the block loads or when an event happens.
- You want to **interact with media previews** or third-party widgets inside the editor.

---

## 💡 Practical Use Cases in Gutenberg Blocks

---

### 1. **Accessing DOM Element to Set Focus**

Let’s say you have a custom input field in your block and want to auto-focus when the block is selected.

```jsx
import { useRef, useEffect } from "@wordpress/element";
import { useSelect } from "@wordpress/data";
import { useBlockSelection } from "@wordpress/block-editor";

export default function Edit({ isSelected }) {
  const inputRef = useRef(null);

  useEffect(() => {
    if (isSelected && inputRef.current) {
      inputRef.current.focus();
    }
  }, [isSelected]);

  return <input ref={inputRef} placeholder="Type here..." />;
}
```

🔍 `isSelected` comes from Gutenberg’s block selection mechanism. This allows the field to be auto-focused when the user clicks on your block.

---

### 2. **Storing Previous Prop or State Values**

Want to compare current and previous block attributes? `useRef` is perfect for this.

```js
const prevUrlRef = useRef();

useEffect(() => {
  if (prevUrlRef.current !== url) {
    console.log("URL has changed!");
    prevUrlRef.current = url;
  }
}, [url]);
```

Use case: This is common when working with images, media uploads, or external data in a block.

---

### 3. **Working with External JS Libraries**

If you integrate a chart, tooltip, or media player, you may need to get the actual DOM node for setup.

```js
const chartContainerRef = useRef();

useEffect(() => {
  const chart = new ChartLibrary(chartContainerRef.current, {
    data: ...
  });

  return () => {
    chart.destroy();
  };
}, []);
```

Use case: Custom Gutenberg blocks with visual libraries (e.g., Chart.js, TinyMCE, Quill, etc.)

---

### 4. **Scroll or Resize Observers**

Use `useRef` to connect with APIs like `ResizeObserver`, `IntersectionObserver`, or to manually scroll an element.

```js
const containerRef = useRef();

useEffect(() => {
  const observer = new ResizeObserver((entries) => {
    console.log("Resized:", entries[0].contentRect);
  });

  if (containerRef.current) {
    observer.observe(containerRef.current);
  }

  return () => observer.disconnect();
}, []);
```

---

### 5. **Avoiding State in Event Handlers**

Sometimes you need to get the **latest value** of a prop or state in an async event handler.

```js
const valueRef = useRef(value);

useEffect(() => {
  valueRef.current = value;
}, [value]);

const handleClick = () => {
  console.log("Latest value is:", valueRef.current);
};
```

This avoids stale closures that can happen inside event listeners or `setTimeout`.

---

## ⚠️ Common Pitfalls

- Don’t **read/write `ref.current` during render**. It should be used in effects or callbacks.
- `useRef` **won’t trigger re-renders**. If you want reactive behavior, use `useState` instead.
- Avoid overusing it for managing data — it's best for referencing **mutable objects or DOM nodes**.

---

## ✅ When to Use `useRef` in Gutenberg Blocks

| Scenario                                     | Use `useRef`?                        |
| -------------------------------------------- | ------------------------------------ |
| Accessing DOM for focus, size, scroll        | ✅ Yes                               |
| Storing mutable values between renders       | ✅ Yes                               |
| Needing values in async handlers             | ✅ Yes                               |
| Triggering UI changes based on stored values | ❌ No — use `useState`               |
| Reading block attributes directly            | ❌ No — attributes are already props |

---

## 🧪 Real Example: MediaUpload Focus and Preview

```js
import { MediaUpload, MediaUploadCheck } from "@wordpress/block-editor";
import { Button } from "@wordpress/components";
import { useRef, useEffect } from "@wordpress/element";

export default function Edit({ attributes, setAttributes, isSelected }) {
  const imageRef = useRef(null);

  useEffect(() => {
    if (isSelected && imageRef.current) {
      imageRef.current.scrollIntoView({ behavior: "smooth" });
    }
  }, [isSelected]);

  return (
    <div>
      <MediaUploadCheck>
        <MediaUpload
          onSelect={(media) => setAttributes({ url: media.url })}
          allowedTypes={["image"]}
          render={({ open }) => <Button onClick={open}>Upload Image</Button>}
        />
      </MediaUploadCheck>

      {attributes.url && (
        <img
          ref={imageRef}
          src={attributes.url}
          alt=""
          style={{ maxWidth: "100%" }}
        />
      )}
    </div>
  );
}
```

Here, `useRef` is used to scroll the image into view when selected — useful in complex editor layouts.

---

## 🧩 Wrapping Up

**`useRef` is your Swiss Army knife** in Gutenberg development when:

- You need DOM control inside your block.
- You want to persist values without causing re-renders.
- You’re building smooth, interactive, editor-first experiences.
