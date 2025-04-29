# 🧩 Gutenberg Block Development with React: Deep Dive into `ToolbarGroup`

If you're building custom blocks in WordPress using React and the Gutenberg editor, you've likely interacted with the block toolbar — **the contextual menu that appears above a block when selected**. In this deep dive, we’re exploring the **`ToolbarGroup`** component: what it is, why it matters, and how you can use it effectively to enhance your block's user interface.

---

## 🔍 What is `ToolbarGroup`?

>`ToolbarGroup` is a **React component** from the `@wordpress/components` package that allows you to **group multiple toolbar controls** within the block toolbar in Gutenberg ❗️
Think of it as a container that lets you **logically organize your toolbar buttons** for cleaner design, better usability, and easier maintenance of your block's UI ❗️

---

## 🎯 Core Purpose of `ToolbarGroup`

- ✅ **Logical Grouping**: Combine related toolbar buttons (like alignments, format toggles, or custom actions) into a single visual group ❗️
- ✅ **Improve UX**: Prevent toolbar clutter by grouping controls under a shared UI space❗️
- ✅ **Extensibility**: Makes it easier for others (or plugins) to hook into your toolbar controls logically❗️
- ✅ **Semantic UI**: Helps convey purpose and grouping, which is important for accessibility (a11y) and screen readers ❗️

---

## ⚙️ Core Concepts and Props

The `ToolbarGroup` component comes with key props and concepts you need to understand:

```js
import { ToolbarGroup } from '@wordpress/components';
```

### 🔧 Props Overview ❗️👇👇👇❗️

| Prop       | Type     | Description                                                                 |
|------------|----------|-----------------------------------------------------------------------------|
| `className`| `string` | Custom class name to style or differentiate the group.                      |
| `children` | `ReactNode` | The toolbar controls you want to group (e.g., `ToolbarButton`).         |
| `isCollapsed` | `boolean` | Determines whether to collapse the group into a dropdown (optional).    |

---

## ✍️ Practical Example: Custom Block with ToolbarGroup

Let’s build a real-world example: A custom block with **bold** and **italic** toggles, grouped under a `ToolbarGroup`.

### 🧱 Step-by-Step Code

```jsx
import { registerBlockType } from '@wordpress/blocks';
import { RichText, BlockControls } from '@wordpress/block-editor';
import { ToolbarGroup, ToolbarButton } from '@wordpress/components';
import { useState } from '@wordpress/element';
import { Bold, Italic } from '@wordpress/icons';

registerBlockType('myplugin/text-toolbar-block', {
  title: 'Toolbar Group Block',
  icon: 'edit',
  category: 'text',
  attributes: {
    content: {
      type: 'string',
      source: 'html',
      selector: 'p',
    },
  },

  edit: ({ attributes, setAttributes }) => {
    const { content } = attributes;
    const [isBold, setBold] = useState(false);
    const [isItalic, setItalic] = useState(false);

    const toggleBold = () => setBold(!isBold);
    const toggleItalic = () => setItalic(!isItalic);

    return (
      <>
        <BlockControls>
          <ToolbarGroup>
            <ToolbarButton
              icon={<Bold />}
              label="Bold"
              isPressed={isBold}
              onClick={toggleBold}
            />
            <ToolbarButton
              icon={<Italic />}
              label="Italic"
              isPressed={isItalic}
              onClick={toggleItalic}
            />
          </ToolbarGroup>
        </BlockControls>

        <RichText
          tagName="p"
          value={content}
          onChange={(value) => setAttributes({ content: value })}
          style={{
            fontWeight: isBold ? 'bold' : 'normal',
            fontStyle: isItalic ? 'italic' : 'normal',
          }}
        />
      </>
    );
  },

  save: ({ attributes }) => {
    return <RichText.Content tagName="p" value={attributes.content} />;
  },
});
```

### ✅ Key Highlights

- All formatting buttons are wrapped inside a single `ToolbarGroup` ❗️💡❗️💡
- We used WordPress icons (`@wordpress/icons`) for the UI.
- The formatting applies dynamically via inline `style`.

---

## 🧠 Advanced Use Cases

### 1. **Conditional ToolbarGroup**
Show or hide a `ToolbarGroup` based on some block state (e.g., if an image is selected or a toggle is enabled).

```jsx
{ showFormatting && (
  <ToolbarGroup>
    <ToolbarButton ... />
    ...
  </ToolbarGroup>
)}
```

### 2. **Collapsible ToolbarGroup**
If you have multiple controls and want to reduce clutter, use `isCollapsed={true}`.

```jsx
<ToolbarGroup isCollapsed>
  <ToolbarButton icon="format-align-left" label="Left" onClick={...} />
  <ToolbarButton icon="format-align-center" label="Center" onClick={...} />
</ToolbarGroup>
```

### 3. **Multiple ToolbarGroups**
Split your UI logic into separate groups (e.g., formatting vs media controls).

```jsx
<BlockControls>
  <ToolbarGroup>
    <ToolbarButton icon="bold" />
  </ToolbarGroup>
  <ToolbarGroup>
    <ToolbarButton icon="image" />
  </ToolbarGroup>
</BlockControls>
```

---

## 🛠️ Styling and Customization

You can apply custom CSS to `ToolbarGroup` using the `className` prop:

```jsx
<ToolbarGroup className="my-custom-toolbar-group">
  ...
</ToolbarGroup>
```

Then in your stylesheet:

```css
.my-custom-toolbar-group {
  background: #f0f0f0;
  border-radius: 4px;
}
```

---

## 🗂️ When to Use `ToolbarGroup`

✅ Use when:
- You have **multiple toolbar buttons** that serve a similar purpose ❗️❗️❗️
- You want to maintain a clean and organized UI ❗️❗️❗️
- You’re building **modular block UIs** that other developers might extend ❗️❗️❗️

❌ Avoid when:
- You only have **one** button (no grouping needed).
- You’re not placing controls in the block toolbar (e.g., Inspector controls instead).

---

## 📌 Conclusion

The `ToolbarGroup` component is a **powerful but often overlooked tool** in your Gutenberg development arsenal. By logically grouping toolbar controls, you improve usability, maintainability, and scalability of your blocks — all while keeping the UI clean and user-friendly.

Whether you’re building formatting tools, layout options, or media controls, **`ToolbarGroup` helps you stay organized without sacrificing flexibility.**

---

## 🚀 Ready to Try It?

Start by identifying which of your toolbar buttons can be grouped. Implement `ToolbarGroup` in your block and see how it enhances your editing experience in Gutenberg.

---

## 🏷️ Suggested Hashtags for Sharing

```markdown
#WordPressDevelopment #GutenbergBlocks #ReactJS #ToolbarGroup #WPDev #BlockEditor #JavaScriptForWP #CustomBlocks #WordPressTips #WPComponents
```
