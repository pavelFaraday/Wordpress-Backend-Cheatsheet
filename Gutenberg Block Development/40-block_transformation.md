# Block Transformation in Gutenberg

Transforming blocks into other blocks in **WordPress Gutenberg (React)** is a powerful feature of the block editor architecture. It allows users to switch one block type to another — either manually via the editor or programmatically via code. This flexibility improves user experience and developer control, particularly when managing custom blocks, patterns, or migrations.

---

## 🎯 Purpose

At its core, block transformation allows:

- **User Experience Enhancements**: Let users easily switch block types (e.g., Paragraph → Heading).
- **Content Reusability**: Allow one block’s content/structure to be reused in another block format.
- **Editor UX Extensions**: Provide tailored transformation options for custom blocks.
- **Migration Paths**: Help evolve block structures without breaking old content.

---

## 🧠 Core Concepts

### 1. **Transforms API**

Gutenberg provides the [`transforms`](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-transforms/) property in a block's registration object. It has two main keys:

```js
transforms: {
  from: [ /* transforms INTO this block */ ],
  to: [ /* transforms FROM this block to another */ ],
}
```

```js
import { registerBlockType, createBlock } from "@wordpress/blocks";
import "./style.scss";
import Edit from "./edit";
import save from "./save";
import metadata from "./block.json";

registerBlockType(metadata.name, {
  icon: {
    src: "text-page",
    foreground: "blue",
    background: "light-gray",
  },
  variations: [
    {
      name: "create-block/gradient-text-box",
      title: "Gradient Text Box",
      description: "A text box with rounded corners.",
      icon: {
        src: "admin-customizer",
        background: "#f0f0f0",
        foreground: "#0073aa",
      },
      attributes: {
        gradient: "red-to-blue",
      },
    },
  ],
  transforms: {
    from: [
      {
        type: "block",
        blocks: ["core/paragraph"],
        transform: ({ content, align }) => {
          return createBlock(metadata.name, {
            text: content,
            alignment: align,
          });
        },
      },
      {
        type: "enter",
        regExp: /textbox/i,
        transform: () => {
          return createBlock(metadata.name, {
            shadow: true,
            gradient: "red-to-blue",
          });
        },
      },
      {
        type: "prefix",
        prefix: "textbox",
        transform: () => {
          return createBlock(metadata.name);
        },
      },
    ],
    to: [
      {
        type: "block",
        blocks: ["core/paragraph"],
        isMatch: ({ text }) => {
          return text ? true : false;
        },
        transform: ({ text, alignment }) => {
          return createBlock("core/paragraph", {
            content: text,
            align: alignment,
          });
        },
      },
    ],
  },
  edit: Edit,
  save,
});
```

---

### 2. **Transform Directions**

- **`from`**: Define how _other blocks_ can be transformed into _your block_.
- **`to`**: Define how _your block_ can be transformed into _other blocks_.

Each transform uses:

```js
{
  type: 'block',
  blocks: ['core/paragraph'], // block name(s)
  transform: (attributes, innerBlocks) => createBlock('your/block-name', { ... })
}
```

---

### 3. **Block Names**

These are typically in the `namespace/block-name` format, e.g., `core/image`, `myplugin/testimonial`.

---

### 4. **Transform Types**

- `block`: One-to-one transformation (most common).
- `raw`: Transform raw HTML into a block.
- `enter`: Used when user types something (like `/textbox`) in the editor.
- `shortcode`: Legacy – convert shortcodes to blocks.

---

## 💡 Practical Examples

### ✅ Basic Example: Paragraph → Custom “TextBox” Block

```js
transforms: {
  from: [
    {
      type: 'block',
      blocks: ['core/paragraph'],
      transform: ({ content }) => {
        return createBlock('myplugin/textbox', {
          text: content,
        });
      },
    },
  ],
}
```

---

### ✅ Adding "to" Transform: Custom → Paragraph

```js
transforms: {
  to: [
    {
      type: 'block',
      blocks: ['core/paragraph'],
      transform: ({ text }) => {
        return createBlock('core/paragraph', {
          content: text,
        });
      },
    },
  ],
}
```

---

### ✅ Enter Transform: Type `/textbox`

```js
{
  type: 'enter',
  regExp: /textbox/i,
  transform: () => {
    return createBlock('myplugin/textbox');
  },
}
```

---

## 🧪 Advanced Use Cases

### 🔁 Transforming Multiple Inner Blocks

If your block uses nested blocks (via `InnerBlocks`), you can carry them over:

```js
transform: ({ title }, innerBlocks) => {
  return createBlock(
    "myplugin/accordion",
    {
      heading: title,
    },
    innerBlocks
  );
};
```

---

### 🧼 Cleaning Attributes During Transform

Sometimes you want to remove or reshape attributes when transforming:

```js
transform: ({ color, content }) => {
  return createBlock("myplugin/clean-block", {
    text: content.trim(),
    style: `color: ${color}`,
  });
};
```

---

## 🛠 Use Cases in Real Projects

### 1. **Custom Design System Blocks**

Transform `core/paragraph` to `custom/heading-with-style` to enforce brand design tokens.

### 2. **Migrate Deprecated Blocks**

Transform `old/block` to `new/block` while updating structure and attributes.

### 3. **Easier Authoring**

Provide `/` command transformations for common custom blocks (e.g., `/faq`, `/testimonial`).

### 4. **Block Variations + Transforms**

Combine `variations` (pre-filled block presets) with `transforms` to offer more intuitive user options.

---

## 🧰 Debugging & Best Practices

- Use `console.log(attributes, innerBlocks)` inside transforms to inspect what you get.
- Always use `createBlock()` — do **not** manually return object structures.
- Ensure `block.json` is properly set up — transforms defined in JS won't show up if block isn't properly registered.

---

## 📘 Resources

- [Gutenberg Block Transforms Reference](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-transforms/)
- [WP Core Example of Transforms](https://github.com/WordPress/gutenberg/tree/trunk/packages/block-library/src)

---

## 🔚 Conclusion

Transforming blocks in Gutenberg is a crucial mechanism for making content creation and management seamless, flexible, and future-proof. Whether you're enabling smoother UX, building a design system, or handling backward compatibility — transforms let you do it elegantly.
