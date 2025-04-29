# Gutenberg Block Development with React: Deep Dive into `ToolbarButton` 🔥

The WordPress Gutenberg editor has revolutionized content creation — and at its heart is **block development**. If you're diving into custom blocks with **React** (and a bit of `@wordpress` magic), one crucial component you'll encounter often is the `ToolbarButton`.

Today, let’s **deep dive** into `ToolbarButton`:  
Its **purpose**, **core concepts**, and **practical use cases** — with real examples to level up your Gutenberg block game. 🚀

---

## 🛠️ What Is `ToolbarButton` in Gutenberg?

At its simplest, **`ToolbarButton`** is a **React component** provided by the Gutenberg packages (specifically, `@wordpress/components`).

>It lets you add **custom buttons** to the **block toolbar** — the small bar above a block where users adjust settings like alignment, bold text, etc ❗️
👉 **In short:**  
`ToolbarButton` = A React button **inside** the block toolbar.

**Purpose:**  
- Add block-specific actions or controls.
- Provide a **native WordPress** feel (consistent UI/UX).
- Improve block editing **efficiency** without cluttering sidebars or inspector panels.

---

## 🔍 Core Concepts of `ToolbarButton`

Let's break down the important parts you need to master:

| Concept | Description |
|:--------|:------------|
| **Importing** | `ToolbarButton` comes from `@wordpress/components`. |
| **Placement** | It's usually nested inside a `BlockControls` wrapper, which places it properly in the toolbar. |
| **Event Handling** | You bind actions (e.g., toggles, modals, styles) to the button’s `onClick` handler ❗️❗️❗️|
| **Visual Customization** | Can include icons, labels, active states, and ARIA accessibility attributes. |
| **State Management** | Often interacts with the block’s `attributes` or internal `state` (via React hooks) ❗️❗️❗️ |

---

## ⚙️ How to Use `ToolbarButton` — Basic Example

Let's start with the **simplest working usage**.

```jsx
import { ToolbarButton } from '@wordpress/components';
import { BlockControls } from '@wordpress/block-editor';
import { useState } from '@wordpress/element';
import { formatBold } from '@wordpress/icons'; // Example icon

export default function Edit({ attributes, setAttributes }) {
    const [ isBold, setIsBold ] = useState( false );

    const toggleBold = () => {
        setIsBold( ( prev ) => !prev );
    };

    return (
        <>
            <BlockControls>
                <ToolbarButton
                    icon={ formatBold }
                    label="Bold Text"
                    isPressed={ isBold }
                    onClick={ toggleBold }
                />
            </BlockControls>

            <p style={ { fontWeight: isBold ? 'bold' : 'normal' } }>
                This is a custom block with a ToolbarButton!
            </p>
        </>
    );
}
```

✅ **What happens:**  
- Adds a bold **toggle** button in the toolbar.
- Clicking the button updates the text weight dynamically.

---

## 🧠 Key Props of `ToolbarButton`

| Prop | Purpose |
|:-----|:--------|
| `icon` | Sets the icon (SVG or WordPress-provided icon) shown inside the button. |
| `label` | Provides accessible text (for screen readers + hover tooltip). |
| `isPressed` | Boolean to show active/pressed styling. Useful for toggles. |
| `onClick` | Callback function triggered when the button is clicked. |
| `isActive` | Alternative to `isPressed`, used when button is conditionally active. |
| `disabled` | If `true`, the button becomes non-interactive. |
| `className` | Allows custom styling classes. |

---

## 🧩 Real World Use Cases of `ToolbarButton`

Here’s where you’ll **actually use** `ToolbarButton` in projects:

### 1. **Toggling Block Styles**
Give users one-click control to bold, italicize, underline, change font size, background, etc.

```jsx
<ToolbarButton
    icon="admin-customizer"
    label="Toggle Background Color"
    onClick={ () => setAttributes({ hasBackground: !attributes.hasBackground }) }
/>
```

---

### 2. **Opening Modals or Popups**
Launch advanced settings, modals, or nested options.

```jsx
<ToolbarButton
    icon="edit"
    label="Open Settings"
    onClick={ openModal }
/>
```
(Then `openModal` triggers a custom `<Modal>` component.)

---

### 3. **Media or Link Insertions**
You can wire it up to media libraries, image uploads, or URL selectors.

Example:  
Button to add/change an image **inside** your block.

```jsx
<ToolbarButton
    icon="format-image"
    label="Change Image"
    onClick={ openMediaSelector }
/>
```

---

### 4. **Switch Between Block Variations**
For example, switching between **list types** (bullet vs. numbered), or **layout options** (grid vs. carousel).

---

## 🚀 Pro Tips for Using `ToolbarButton` Like a Pro

- **Keep it lightweight:** Only show toolbar buttons that are **essential** to your block’s operation.
- **Group logically:** If you have multiple buttons, group them meaningfully using `ToolbarGroup`.
- **Accessibility matters:** Always include `label` props for screen readers.
- **Use WordPress icons:** Import from `@wordpress/icons` to maintain visual consistency with WordPress UI.
- **Combine with Inspector Controls:** Sometimes a `ToolbarButton` is for quick toggles, and **full settings** live in the sidebar.

---

## 📚 Additional Resources

- [ToolbarButton Reference (WordPress Developer Docs)](https://developer.wordpress.org/block-editor/reference-guides/components/toolbar-button/)
- [@wordpress/components GitHub](https://github.com/WordPress/gutenberg/tree/trunk/packages/components/src/toolbar-button)
- [Gutenberg Handbook: Block Controls](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-controls/)

---

# 🎯 Conclusion

Mastering `ToolbarButton` unlocks a **professional-level editing experience** in your custom Gutenberg blocks.  
It’s lightweight, extensible, and **makes your blocks feel native** inside WordPress.

If you're building **interactive blocks**, **user-friendly toolbars**, or just want to **polish your plugin**, learning how to **strategically implement ToolbarButton** is absolutely worth it. 💥

