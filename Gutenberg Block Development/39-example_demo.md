# 🔍 What is `example` in Gutenberg?

In the context of Gutenberg block development, the `example` key is a **static representation of a block's default or demo state**, used **specifically for previews inside the block inserter** (a.k.a. block picker). It allows developers to **show a filled-out version** of the block before it’s inserted — making it visually understandable to users.

---

## 🎯 Purpose of the `example` Property

The `example` property in Gutenberg serves several purposes:

### ✅ 1. **Visual Preview**

When users browse blocks in the inserter, the `example` provides a **preview image or live JSX rendering** of how the block will look.

### ✅ 2. **Enhanced UX**

It gives users **better insight into the block's purpose** without inserting it blindly and having to configure it.

### ✅ 3. **Better Onboarding**

Especially important for **custom blocks**, where content or layout might not be obvious without context.

---

## 🧠 Core Concepts

### 🧩 Where it is defined?

The `example` key is defined inside `registerBlockType()`:

```js
registerBlockType("namespace/block-name", {
  title: "My Custom Block",
  category: "widgets",
  icon: "smiley",
  attributes: {
    content: { type: "string" },
  },
  edit,
  save,
  example: {
    attributes: {
      content: "Example text goes here!",
    },
  },
});
```

### 🧬 How it works

- The editor detects the `example` property.
- It renders the block in a **"preview" mode** with the defined attributes.
- If `example` is omitted, the block shows a generic or blank preview.

---

## 💡 Practical Use Cases

### 📦 1. **Default Content Preview**

You want your block to show how it behaves with real content.

```js
example: {
  attributes: {
    heading: 'Hello from the example!',
    body: 'This is example content shown in the inserter preview.',
  },
}
```

### 🖼️ 2. **Block Preview with Images or Media**

If your block contains media or images:

```js
example: {
  attributes: {
    imageURL: 'https://via.placeholder.com/150',
    caption: 'Preview Caption',
  },
}
```

### 🧰 3. **Complex Nested Blocks**

If your block uses `InnerBlocks`, Gutenberg lets you preview with nested blocks:

```js
example: {
  innerBlocks: [
    {
      name: 'core/paragraph',
      attributes: { content: 'Inside a nested block!' },
    },
  ],
}
```

---

## ⚠️ Best Practices

1. **Keep it lightweight** — don’t add API calls or dynamic content in examples.
2. **Use real-looking placeholder text** — helps with context.
3. **Avoid logic in previews** — `example` should be static.
4. **Fallback for no preview** — handle `isSelected` or `context` for fallback rendering.

---

## 🧪 Advanced: Conditional Preview vs Edit Mode

Inside your `edit` function, you can detect when the block is being rendered as a preview (using the `useBlockPreviewContext` hook in newer Gutenberg versions or `props.context` in older ones):

```js
import { useBlockPreviewContext } from "@wordpress/block-editor";

const Edit = (props) => {
  const previewContext = useBlockPreviewContext();
  const isPreview = previewContext && previewContext.isPreview;

  if (isPreview) {
    return <div>This is the preview view</div>;
  }

  return (
    <div>
      <RichText
        tagName="p"
        value={props.attributes.content}
        onChange={(value) => props.setAttributes({ content: value })}
      />
    </div>
  );
};
```

---

## 🛠️ Example in Full Block Registration

```js
registerBlockType("myplugin/example-block", {
  title: "Example Block",
  icon: "admin-site",
  category: "design",
  attributes: {
    content: {
      type: "string",
      default: "",
    },
  },
  edit: ({ attributes, setAttributes }) => (
    <RichText
      tagName="p"
      value={attributes.content}
      onChange={(value) => setAttributes({ content: value })}
      placeholder="Enter some text..."
    />
  ),
  save: ({ attributes }) => (
    <RichText.Content tagName="p" value={attributes.content} />
  ),
  example: {
    attributes: {
      content: "Previewed example content!",
    },
  },
});
```

---

## 📌 Summary

| Feature          | Purpose                                            |
| ---------------- | -------------------------------------------------- |
| `example` key    | Provides static preview data                       |
| Used in          | `registerBlockType()`                              |
| Renders in       | Block inserter (preview mode)                      |
| Supports         | `attributes`, `innerBlocks`, `viewportWidth`       |
| Enhances         | UX, onboarding, clarity before insertion           |
| Avoid in preview | Dynamic content, API fetches, or conditional logic |

---

## ✅ Final Tip

If you're building commercial or complex blocks — **invest time in crafting great `example` data**. It directly impacts the **user experience** and **perceived quality** of your block.
