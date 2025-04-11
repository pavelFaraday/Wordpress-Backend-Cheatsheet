# 🚀 Gutenberg Block Development with React: `useInnerBlocksProps` & `InnerBlocks`

If you're diving into **custom Gutenberg block development**, chances are you've encountered `InnerBlocks` and maybe even the `useInnerBlocksProps` hook. These tools are essential when you want your block to act as a **container for other blocks** — like a section, column, or card layout.

In this article, we’ll **deep dive** into both, breaking down:
- What they are
- How they work
- When to use which
- Practical code examples

---

## 🧠 Understanding the Purpose

### 👉 What is `InnerBlocks`?

`InnerBlocks` is a **Gutenberg component** that allows your custom block to **nest other blocks inside it**. Think of it like `children` in React — it provides a slot where other blocks can live.

> This is perfect when building reusable containers like cards, sections, columns, tabs, or accordions.

### 👉 What is `useInnerBlocksProps`?

Introduced later for **better integration with React and JSX**, `useInnerBlocksProps` is a **React hook** that connects your container element to the Gutenberg editor and sets it up to accept nested blocks.

It replaces the need for `<InnerBlocks />` in some advanced use cases, especially where **you want to control or enhance the wrapper element** around the `InnerBlocks`.

---

## 🛠 Core Concepts & Differences

| Feature | `InnerBlocks` | `useInnerBlocksProps` |
|--------|----------------|-----------------------|
| Type | Component | React Hook |
| Use Case | Basic usage, simple containers | Custom wrappers or advanced layout control |
| Return Value | JSX component | Props object for your wrapper element |
| Common Use | Columns, groups, tabs, cards | Advanced containers, semantic elements (e.g., `<section>`, `<ul>`) |

---

## 💡 When Should You Use Each?

| Use This | When You Want To |
|----------|------------------|
| `InnerBlocks` | Quickly add nested blocks inside your custom block. |
| `useInnerBlocksProps` | Add props/styles/handlers to a wrapper element and still use `InnerBlocks`. Perfect for structured HTML like `<div class="card"><div class="card-body">[InnerBlocks]</div></div>`. |

---

## 🧑‍💻 Example 1: Basic Usage with `InnerBlocks`

Let’s say we’re building a **simple container block**.

```jsx
// edit.js
import { InnerBlocks } from '@wordpress/block-editor';

const Edit = () => {
  return (
    <div className="custom-container-block">
      <InnerBlocks
        allowedBlocks={['core/paragraph', 'core/image']}
        template={[['core/paragraph', { placeholder: 'Write something...' }]]}
        templateLock={false}
      />
    </div>
  );
};

export default Edit;
```

💡 Use this when:
- You don't need extra control over the wrapper element.
- A simple div with editable content is sufficient.

---

## 🧑‍💻 Example 2: Advanced Wrapper with `useInnerBlocksProps`

Now let’s say we’re building a **custom card block** that requires custom classes and styles.

```jsx
// edit.js
import { useBlockProps, useInnerBlocksProps } from '@wordpress/block-editor';

const Edit = () => {
  const blockProps = useBlockProps({ className: 'custom-card' });
  const innerBlocksProps = useInnerBlocksProps(
    {
      className: 'custom-card-content',
    },
    {
      allowedBlocks: ['core/paragraph', 'core/heading'],
      template: [['core/heading', { level: 3, placeholder: 'Card Title' }]],
    }
  );

  return (
    <div {...blockProps}>
      <div {...innerBlocksProps} />
    </div>
  );
};

export default Edit;
```

💡 Use this when:
- **You want control over both the outer and inner structure** ❗️❗️❗️
- **You need to attach styles/classes or handle more complex layouts** ❗️❗️❗️

---

## 🧩 Saving the Block: `InnerBlocks.Content`

No matter which method you use in the edit function, in your `save.js` file, you’ll typically use:

```jsx
import { useBlockProps, InnerBlocks } from '@wordpress/block-editor';

const Save = () => {
  return (
    <div {...useBlockProps.save()}>
      <InnerBlocks.Content />
    </div>
  );
};

export default Save;
```

This tells Gutenberg how to save the nested content to the database.

---

## 🧪 Real Use Case: Custom Grid Block with Inner Blocks

Imagine a block that represents a **grid layout** where each cell is an InnerBlock.

### Structure:
- Grid Block (parent)
- Cell Block (child with `InnerBlocks` or `useInnerBlocksProps`)

This allows end users to create reusable layouts while you maintain control over structure and styling.

---

## 📝 Summary

| Concept | Use It When |
|--------|-------------|
| `InnerBlocks` | You need a quick way to let users add nested blocks. |
| `useInnerBlocksProps` | You want **fine control** over how the nested blocks render or are wrapped. |

### ✅ Best Practices
- Always wrap `InnerBlocks` with `useBlockProps` or `useInnerBlocksProps`.
- Define `allowedBlocks` and `template` for better UX.
- Use `templateLock` if you want to restrict layout changes.

---

## 🔚 Final Thoughts

`useInnerBlocksProps` and `InnerBlocks` give you **powerful flexibility** in building container-style Gutenberg blocks. Mastering these will unlock a whole new level of customization in your WordPress development workflow.

---

## 💬 What’s Next?

Are you planning to build a block layout with nested content? Need help choosing between `InnerBlocks` or `useInnerBlocksProps`?

Let me know in the comments! 💬👇

---

## 🔖 Suggested Hashtags (For LinkedIn or Twitter)
```
#WordPressDevelopment #GutenbergBlocks #ReactJS #InnerBlocks #WebDevelopment #FullStack #WPGutenberg #BlockEditor #WordPressTips
```