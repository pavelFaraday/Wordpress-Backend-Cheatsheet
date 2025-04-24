# Gutenberg Block Development with React: Deep Dive into `JustifyContentControl`

If you’ve ever worked on custom Gutenberg blocks and needed layout flexibility, chances are you’ve stumbled upon the `JustifyContentControl` component. This hidden gem in WordPress block development provides an intuitive way to control content alignment—right from your block’s sidebar settings.

But what exactly is it? How does it work? And when should you use it?

Let’s dive deep.

---

## 🎯 What is `JustifyContentControl`?

`JustifyContentControl` is a component provided by the `@wordpress/block-editor` package. It's used to define and control the horizontal alignment (justification) of inner block content—typically when using the `InnerBlocks` component.

This control maps directly to the CSS property `justify-content`, which is essential for flexbox layouts.

### 🔍 Importing the Component

```js
import { JustifyContentControl } from '@wordpress/block-editor';
```

---

## 🧠 Core Concepts

To really grasp `JustifyContentControl`, you need to understand a few key concepts in Gutenberg and React:

### 1. **InnerBlocks + Layouts**
Most use cases for `JustifyContentControl` come when your block includes `InnerBlocks`—essentially a container block. This is where layout management matters.

```jsx
<InnerBlocks
  allowedBlocks={['core/paragraph', 'core/image']}
  orientation="horizontal"
  templateLock={false}
/>
```

When wrapping these blocks, you often use a flex container, and here’s where the justification matters.

### 2. **Flexbox and justify-content**
`JustifyContentControl` is designed to abstract `justify-content` values like:

- `flex-start`
- `center`
- `flex-end`
- `space-between`
- `space-around`
- `space-evenly`

These directly control how child elements (InnerBlocks) align horizontally within the container.

### 3. **User-Friendly UI**
Instead of expecting users to know or write flexbox rules, `JustifyContentControl` gives them an icon-based toolbar (usually in the block inspector), allowing easy alignment control.

---

## 🛠️ Practical Implementation

Let’s look at a working example inside a custom block.

### 1. Define the Block Attributes

```js
attributes: {
  justifyContent: {
    type: 'string',
    default: 'center',
  },
}
```

### 2. Add the Control in `edit.js`

```jsx
import { InspectorControls, JustifyContentControl } from '@wordpress/block-editor';
import { PanelBody } from '@wordpress/components';

const Edit = (props) => {
  const { attributes, setAttributes } = props;
  const { justifyContent } = attributes;

  return (
    <>
      <InspectorControls>
        <PanelBody title="Layout Settings">
          <JustifyContentControl
            label="Horizontal Alignment"
            value={justifyContent}
            onChange={(newVal) => setAttributes({ justifyContent: newVal })}
          />
        </PanelBody>
      </InspectorControls>

      <div
        className="my-custom-container"
        style={{ display: 'flex', justifyContent }}
      >
        <InnerBlocks />
      </div>
    </>
  );
};
```

### 3. Save Function

In `save.js`, reflect the selected alignment:

```jsx
export default function save({ attributes }) {
  const { justifyContent } = attributes;

  return (
    <div
      className="my-custom-container"
      style={{ display: 'flex', justifyContent }}
    >
      <InnerBlocks.Content />
    </div>
  );
}
```

---

## 💼 Real-World Use Cases

### 🧱 Custom Section Blocks
Use `JustifyContentControl` when building section blocks like hero areas or call-to-action sections that contain multiple blocks and require flexible layout options.

### 📐 Grid or Row Layout Blocks
In blocks where items are displayed horizontally—like cards, columns, or icon lists—adding horizontal alignment through this control empowers non-developers to customize layout visually.

### 💬 Testimonial Blocks
Need to center-align testimonials or justify them to the end of the row? This control makes it effortless.

---

## ⚠️ Common Pitfalls

- **Not applying `display: flex`**: The selected alignment won't work unless the container has `display: flex`.
- **Missing save attributes**: Always ensure your saved block output reflects the same `justify-content` logic as in the editor.
- **Responsiveness**: `JustifyContentControl` doesn’t handle responsive breakpoints out of the box. You’ll need additional logic or classes if alignment needs to change per screen size.

---

## 🧩 Alternatives & Extensions

- Pair it with `AlignmentControl` for text alignment.
- Combine with `useBlockProps` for cleaner integration.
- Extend it with custom UI if you need more than what `JustifyContentControl` offers.

---

## 🚀 Final Thoughts

`JustifyContentControl` may seem like a small piece of the Gutenberg puzzle, but it offers a huge UX win when building layout-oriented blocks. By letting users control alignment visually—and without touching CSS—you make your blocks truly user-friendly and editor-native.

Whether you're building advanced flex layouts or simple row displays, don’t overlook this control in your custom block toolbox.

---

## 📌 Let’s Wrap It Up

If you’re creating Gutenberg blocks with React and want to offer layout control to users, `JustifyContentControl` is a must-know tool. Flexible, intuitive, and directly tied to CSS best practices—it brings clarity and polish to your block’s user experience.

---