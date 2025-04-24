# 🔄 Gutenberg Block Development with React: Deep Dive into `ToggleControl`

If you're building custom Gutenberg blocks for WordPress, you've likely come across `ToggleControl`. It’s one of the most intuitive components provided by `@wordpress/components`—yet many developers overlook its full potential.

In this deep dive, we’ll unpack everything about `ToggleControl`: its **purpose**, **core concepts**, and **how to use it effectively in real-world blocks**.

---

## ✅ What is `ToggleControl`?

The `ToggleControl` is a UI component in the Gutenberg block editor that lets users **enable or disable a setting using a simple switch (on/off). It’s basically a fancy checkbox styled as a toggle** ❗️❗️❗️

### 📦 Import Path

```js
import { ToggleControl } from '@wordpress/components';
```

### 🧠 When to Use It?

Use `ToggleControl` when you want to give block users the ability to switch a **binary setting**—like **showing or hiding an element, turning a feature on/off, or enabling a style toggle** ❗️❗️❗️

Examples:
- Show/hide image captions ❗️
- Enable/disable box shadows ❗️
- Toggle dark mode for a block ❗️

---

## 🧩 Anatomy of `ToggleControl`

Here’s a breakdown of its most important props:

```jsx
<ToggleControl
  label="Enable Dark Mode"
  checked={attributes.enableDarkMode}
  onChange={(value) => setAttributes({ enableDarkMode: value })}
/>
```

### 🧷 Props Explained

| Prop         | Type       | Description |
|--------------|------------|-------------|
| `label`      | `string`   | The visible label beside the toggle switch. |
| `checked`    | `boolean`  | The current state of the toggle (true = on). |
| `onChange`   | `function` | Callback triggered when the toggle is clicked. |

---

## 🛠️ Practical Use Case: Add ToggleControl to a Custom Block

Let’s build a custom block that shows or hides a quote based on a toggle.

### 1. Register the Block

```js
import { registerBlockType } from '@wordpress/blocks';
import { __ } from '@wordpress/i18n';

registerBlockType('myplugin/quote-toggle', {
  title: __('Toggle Quote Block', 'myplugin'),
  icon: 'format-quote',
  category: 'common',
  attributes: {
    showQuote: {
      type: 'boolean',
      default: true,
    },
  },
  edit: EditComponent,
  save: SaveComponent,
});
```

### 2. Create the Edit Component

```jsx
import { ToggleControl } from '@wordpress/components';
import { useBlockProps } from '@wordpress/block-editor';

const EditComponent = ({ attributes, setAttributes }) => {
  const { showQuote } = attributes;

  return (
    <div {...useBlockProps()}>
      <ToggleControl
        label="Show Quote"
        checked={showQuote}
        onChange={(value) => setAttributes({ showQuote: value })}
      />
      {showQuote && <blockquote>“Simplicity is the ultimate sophistication.”</blockquote>}
    </div>
  );
};
```

### 3. Create the Save Component

```jsx
import { useBlockProps } from '@wordpress/block-editor';

const SaveComponent = ({ attributes }) => {
  const { showQuote } = attributes;

  return (
    <div {...useBlockProps.save()}>
      {showQuote && <blockquote>“Simplicity is the ultimate sophistication.”</blockquote>}
    </div>
  );
};
```

---

## 🔍 Real-World Examples

Here are a few ways you can use `ToggleControl` in production:

### 📷 Show/Hide Image Caption

```jsx
<ToggleControl
  label="Show Image Caption"
  checked={attributes.showCaption}
  onChange={(value) => setAttributes({ showCaption: value })}
/>
```

### 🎨 Enable Custom Styling

```jsx
<ToggleControl
  label="Enable Border Radius"
  checked={attributes.hasRadius}
  onChange={(value) => setAttributes({ hasRadius: value })}
/>
```

### 🕶️ Toggle Dark/Light Theme

```jsx
<ToggleControl
  label="Use Dark Theme"
  checked={attributes.darkTheme}
  onChange={(value) => setAttributes({ darkTheme: value })}
/>
```

---

## 🧠 Pro Tips

- **Naming**: Use meaningful labels like "Show Content" instead of generic ones like "Enable Option".
- **Conditional Rendering**: Always pair `ToggleControl` with a conditional render for visible changes.
- **State Syncing**: Make sure `checked` is always tied to a reliable state (usually from `attributes`).
- **Combine with Panels**: For a cleaner UI, place `ToggleControl` inside a `PanelBody` from `@wordpress/components`.

---

## 🚀 Final Thoughts

`ToggleControl` may seem simple, but it’s a powerful UI tool for enhancing the editing experience in Gutenberg. With clean toggles, you’re giving users intuitive controls—no clutter, just smart interactivity.

Whether it’s for content visibility, feature flags, or stylistic toggles, `ToggleControl` helps keep your blocks user-friendly and professional.

---

## 💬 Over to You!

Have you used `ToggleControl` in your Gutenberg blocks? What’s your most creative use of it?

Drop your thoughts below 👇  
Or share this article with a dev friend who’s diving into Gutenberg block development.

---
