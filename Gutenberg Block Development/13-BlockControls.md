# Gutenberg Block Development with React: Deep Dive into `BlockControls` ⚙️

If you’re building custom Gutenberg blocks using React, you’ve likely encountered `BlockControls`. But what *exactly* is it? How does it work? And when should you use it?

This article takes a deep dive into the `BlockControls` component, exploring its **purpose**, **core concepts**, and **real-world use cases**—giving you the tools to supercharge your custom WordPress block development.

---

## 🚀 What is `BlockControls`?

In Gutenberg, `BlockControls` is a React component provided by `@wordpress/block-editor`. 
❗️❗️❗️It allows you to add **toolbar controls** to the block toolbar that appears when a block **is selected**.❗️❗️❗️

### Why use `BlockControls`?

Because it’s the **perfect place** to include settings that are:

- Contextual to the block’s appearance or behavior❗️
- Not suited for the right sidebar (Inspector Controls) ❗️
- Commonly found in formatting toolbars (**alignment, toggles**, etc.)❗️

---

## 🔧 Core Concepts Behind `BlockControls`

Here’s what makes `BlockControls` tick:

### 1. Toolbar Placement 🎯

`BlockControls` renders controls in the toolbar **above** the block—this makes it ideal for **inline formatting tools** (think alignment buttons or toggles).

### 2. Conditional Rendering Based on Focus 👁️

**It only appears when the block is selected ❗️, keeping the UI clean and non-intrusive.**

### 3. Used Inside `edit()` Function 🔁

> **It’s always defined within the `edit` function of your custom block. It’s not used in the `save` function because it’s only relevant in the editor experience.** ❗️❗️❗️

```js
import { BlockControls, AlignmentToolbar } from '@wordpress/block-editor';
```

---

## 🛠️ Practical Example: Text Alignment for a Custom Paragraph Block

Let’s walk through a simple implementation.

```js
import { registerBlockType } from '@wordpress/blocks';
import { RichText, BlockControls, AlignmentToolbar } from '@wordpress/block-editor';
import { useState } from '@wordpress/element';

registerBlockType('myplugin/custom-paragraph', {
  title: 'Custom Paragraph',
  icon: 'edit',
  category: 'common',
  attributes: {
    content: { type: 'string', source: 'html', selector: 'p' },
    alignment: { type: 'string', default: 'left' },
  },

  edit({ attributes, setAttributes }) {
    const { content, alignment } = attributes;

    const onChangeContent = (newContent) => setAttributes({ content: newContent });
    const onChangeAlignment = (newAlign) => setAttributes({ alignment: newAlign });

    return (
      <>
        <BlockControls>
          <AlignmentToolbar value={alignment} onChange={onChangeAlignment} />
        </BlockControls>
        <RichText
          tagName="p"
          value={content}
          onChange={onChangeContent}
          style={{ textAlign: alignment }}
        />
      </>
    );
  },

  save({ attributes }) {
    const { content, alignment } = attributes;

    return <p style={{ textAlign: alignment }}>{content}</p>;
  },
});
```

### 🔍 Key Parts:

- **`BlockControls` wraps the toolbar components ❗️**
- `AlignmentToolbar` provides **left, center, right alignment** options ❗️
- The selected alignment value is saved as a block attribute.

---

## 💼 Use Cases in Real-World Projects

You can use `BlockControls` for a range of settings:

### ✅ Common Use Cases

| Use Case                   | Component Example              |
|----------------------------|-------------------------------|
| Text alignment             | `<AlignmentToolbar />`        |
| Button styles              | `<ToolbarButton />`           |
| Media settings (flip, crop)| Custom `<ToolbarGroup />`     |
| Toggle switches            | `<ToolbarToggleButton />`     |
| Dropdown options           | `<DropdownMenu />`            |

### 🎯 When NOT to use `BlockControls`:

Use `InspectorControls` instead if:

- The setting is **global** (not contextual) ❗️
- It affects **overall** layout or appearance ❗️
- It requires more complex input (text fields, sliders, etc.)

---

## 🧪 Advanced Tip: Grouping Multiple Toolbar Controls

You can group multiple controls like this:

```jsx
<BlockControls>
  <ToolbarGroup>
    <ToolbarButton icon="format-bold" onClick={toggleBold} />
    <ToolbarButton icon="format-italic" onClick={toggleItalic} />
  </ToolbarGroup>
</BlockControls>
```

Use `<ToolbarGroup>` to create a cohesive toolbar experience.

---

## 📌 Final Thoughts

The `BlockControls` component is an essential tool in your Gutenberg development toolkit. It enhances the **editor experience** by giving users intuitive access to **formatting and settings directly on the block**. It’s clean, contextual, and powerful.

Whether you're adjusting alignment or building a custom formatting toolbar, understanding how to properly use `BlockControls` will make your blocks more user-friendly and professional.

---

### 💬 Let’s Chat!

Are you using `BlockControls` in an interesting way? Got a tricky UX challenge you’re trying to solve in a custom block?

Drop your thoughts below 👇 or connect with me for more Gutenberg tips!

---

### 🔖 Recommended Hashtags for Sharing

`#WordPressDev` `#Gutenberg` `#ReactJS` `#BlockEditor` `#WPDeveloper` `#FullSiteEditing` `#WordPressBlocks` `#GutenbergBlocks` `#JavaScript`

---