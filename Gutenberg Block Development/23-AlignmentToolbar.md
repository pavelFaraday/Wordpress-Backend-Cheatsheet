# Gutenberg Block Development with React: Deep Dive into `AlignmentToolbar` 🔥

In the evolving world of WordPress Gutenberg block development, **`AlignmentToolbar`** is a small but mighty tool when you're building flexible, polished blocks.  
Today, we're going deep: you'll understand its **purpose**, **core concepts**, and **real-world use cases** — and yes, we'll get our hands dirty with **practical React examples**.

---

## 🧠 What is `AlignmentToolbar`?

>`AlignmentToolbar` is a **UI component** provided by `@wordpress/block-editor`.  
It renders a **group of alignment buttons** (like left, center, right, wide, full-width) inside the **block toolbar** — helping users align block content without writing custom UIs. So, It makes it dead simple to offer **content alignment control** inside your block!

---

## 🎯 Core Purpose

- ✅ Offer a **native** WordPress experience for alignment settings.
- ✅ Ensure **consistency** across different Gutenberg blocks.
- ✅ Allow users to quickly control **text** or **block** alignment visually.
- ✅ Save you from having to **rebuild** alignment logic manually.

When you use `AlignmentToolbar`, your block instantly looks and behaves like core WordPress blocks (like Paragraph, Image, Heading).

---

## 📚 Core Concepts You Must Know

| Concept | Explanation |
|:---|:---|
| **BlockControls** | A toolbar that appears when a block is selected. You insert the `AlignmentToolbar` *inside* `BlockControls` ❗️ |
| **onChange Handler** | You must connect `AlignmentToolbar` to your block attributes via an `onChange` function❗️ |
| **Alignment Options** | Supports alignments like `'left'`, `'center'`, `'right'`, `'wide'`, `'full'` ❗️ |
| **Dynamic Rendering** | Your block's **visual output** should change based on the selected alignment. |

---

## ⚙️ How `AlignmentToolbar` Works in Practice

Let’s **build a simple example**: a custom block that displays a heading you can align using `AlignmentToolbar`.

---

### Step 1: Setup Your Block Attributes

```javascript
// block.json
{
  "attributes": {
    "content": {
      "type": "string",
      "default": "Hello Gutenberg!"
    },
    "alignment": {
      "type": "string",
      "default": "center"
    }
  }
}
```

We have two attributes:  
- `content` – the heading text.
- `alignment` – the alignment (left, center, right, etc.).

---

### Step 2: Create the Edit Function with `AlignmentToolbar`

```javascript
// edit.js
import { useBlockProps, BlockControls, AlignmentToolbar } from '@wordpress/block-editor';
import { RichText } from '@wordpress/block-editor';

export default function Edit({ attributes, setAttributes }) {
  const { content, alignment } = attributes;

  const blockProps = useBlockProps({
    className: `has-text-align-${alignment}`
  });

  return (
    <>
      <BlockControls>
        <AlignmentToolbar
          value={alignment}
          onChange={(newAlignment) => setAttributes({ alignment: newAlignment })}
        />
      </BlockControls>

      <div {...blockProps}>
        <RichText
          tagName="h2"
          value={content}
          onChange={(newContent) => setAttributes({ content: newContent })}
          placeholder="Write heading..."
          allowedFormats={[]}
        />
      </div>
    </>
  );
}
```

🔵 **Key Points Here:**
- `BlockControls` wraps toolbar controls shown when block is selected.
- `AlignmentToolbar` shows alignment buttons (left/center/right/etc.).
- `onChange` updates the `alignment` attribute dynamically.

---

### Step 3: Handle the Alignment on the Frontend

```javascript
// save.js
import { useBlockProps, RichText } from '@wordpress/block-editor';

export default function save({ attributes }) {
  const { content, alignment } = attributes;

  const blockProps = useBlockProps.save({
    className: `has-text-align-${alignment}`
  });

  return (
    <div {...blockProps}>
      <RichText.Content tagName="h2" value={content} />
    </div>
  );
}
```

✅ This ensures when the block is rendered on the actual site, it keeps the correct alignment.

WordPress already provides CSS classes like `.has-text-align-center`, `.has-text-align-left`, etc., by default!

---

## 🔥 Real-World Use Cases

Here’s where you should use `AlignmentToolbar`:

- **Text Blocks:** Let users align text left, center, or right.
- **Image or Media Blocks:** Allow media to align inside the page layout.
- **Custom Layout Blocks:** Provide control over wide/full width layouts.
- **Reusable Components:** Anytime a subcomponent (like a callout or button) needs alignment flexibility.

---

## ⚡ Advanced Tips

- **Conditional Display:** You can control which alignment options appear using `AlignmentToolbar` props like `controls`.
  
  Example:

  ```javascript
  <AlignmentToolbar
    value={alignment}
    onChange={(newAlignment) => setAttributes({ alignment: newAlignment })}
    controls={['left', 'center', 'right']}
  />
  ```

- **Combine with `ToolbarGroup`:** If you have multiple controls, group them neatly using `ToolbarGroup`.

- **Custom Styling:** For blocks that aren’t just text (e.g., cards or custom layouts), you might manually manage alignment using flexbox/grid and update styles dynamically based on the selected alignment.

---

## 📈 Why Use `AlignmentToolbar` Instead of a Custom Solution?

| `AlignmentToolbar` Advantage | DIY Custom Alignment |
|:---|:---|
| ✔️ Native WordPress UX | ❌ May look inconsistent |
| ✔️ Accessibility ready | ❌ May miss a11y features |
| ✔️ Automatic CSS classes support | ❌ Need to manually style |
| ✔️ Easier maintenance & updates | ❌ More technical debt |

---

## 🚀 Quick Recap

- `AlignmentToolbar` is essential for **alignment control** in WordPress Gutenberg blocks.
- It lives inside **BlockControls** and is connected to a block attribute ❗️❗️❗️
- Always ensure your block's **frontend** honors the selected alignment.
- Use it to **boost UX**, **save development time**, and **stay consistent** with the WordPress ecosystem.

---

# 🏆 Final Thoughts

If you're serious about building **professional-grade Gutenberg blocks**, mastering components like `AlignmentToolbar` is a must.  
It’s a small addition — but it **elevates** your block’s polish and usability massively.

> **Next Step:**  
> Try adding `AlignmentToolbar` to one of your existing custom blocks and watch how much better the user experience becomes! 🔥
