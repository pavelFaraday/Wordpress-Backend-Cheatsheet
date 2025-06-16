# Gutenberg Block Development: The Key Difference Between `name` and `title` `in block.json`🧩

When building custom Gutenberg blocks in WordPress using React, the `block.json` file is your block's central configuration hub. Two of the most commonly misunderstood properties in this file are:

- `name`
- `title`

They **look similar**, but they serve **entirely different purposes**. If you're diving into Gutenberg development, especially using React and modern JavaScript tooling, understanding the distinction is critical.

Let’s break it down.

---

## 🔑 `name`: The Block’s <u>Internal Identifier</u>

### 📌 What It Is:

❗️❗️❗️ The `name` is a <u>**unique machine-readable identifier**</u> used internally by WordPress to register and reference your block.
❗️❗️❗️ When you call `registerBlockType`, WordPress looks at the `name` ❗️❗️❗️
❗️❗️❗️ The `name` is for **WordPress and developers**.


### 📋 Syntax Example:

```json
{
  "name": "myplugin/awesome-block"
}
```

This value must be **namespaced**, using the format:

```
namespace/block-name
```

Where:

- `namespace` = Your plugin or project name (`myplugin`)
- `block-name` = A lowercase, hyphenated identifier (`awesome-block`)

### 🔧 Purpose & Important Notes ⚠️ :

- ❗️❗️❗️ It’s how Gutenberg **registers** the block internally.
- ❗️❗️❗️ It’s how you refer to the block in JavaScript code (e.g., when using `InnerBlocks` or `allowedBlocks`).
- ❗️❗️❗️ It **must be unique** across all registered blocks to avoid conflicts.
- ❗️❗️❗️ This is **not shown** in the WordPress UI.
- ❗️❗️❗️ Changing the `name` after the block is in use can **break** content that uses the block.
- ❗️❗️❗️ The `name` **must match** the identifier used in `registerBlockType`.

### ✅ Use Case Example:

```js
registerBlockType("myplugin/awesome-block", {
  edit: EditComponent,
  save: SaveComponent,
});
```

---

## 🏷️ `title`: The Block’s <u>Display Name</u>

### 📌 What It Is:

❗️❗️❗️ The `title` is the **human-readable label** shown in the WordPress block inserter (the block picker panel).
❗️❗️❗️ The `title` is for **content creators and users**.

### 📋 Syntax Example:

```json
{
  "title": "Awesome Block"
}
```

### 💡 Purpose & Practical Impact 🛠:

- ❗️❗️❗️ This is what **content creators see** when adding blocks in the editor.
- ❗️❗️❗️ It supports **localization**, allowing you to provide translations.
- ❗️❗️❗️ You can make it descriptive and even marketing-friendly for non-dev users.
- ❗️❗️❗️ Good `title` helps users find and recognize your block easily.
- ❗️❗️❗️ It's searchable in the block inserter by this title.

### ✅ Use Case Example:

```json
{
  "title": "Call to Action Banner"
}
```

**This will be shown in the block inserter as a block titled "Call to Action Banner".**

---

## 🔍 Summary: `name` vs `title`

| Property | Purpose                   | Visible to Users? | Format                 | Changeable After Use? |
| -------- | ------------------------- | ----------------- | ---------------------- | --------------------- |
| `name`   | <u>Internal ID for the block</u> | ❌ No             | `namespace/block-name` | ⚠️ Not recommended    |
| `title`  | <u>Display name in editor UI</u> | ✅ Yes            | Human-readable string  | ✅ Safe to change     |

---

## 🧪 Real-World Example

**block.json:**

```json
{
  "name": "myplugin/testimonial-slider",
  "title": "Testimonial Slider",
  "category": "widgets",
  "icon": "format-quote",
  "description": "A stylish slider to showcase testimonials.",
  "supports": {
    "html": false
  },
  "editorScript": "file:./index.js"
}
```

**How it works in practice:**

- ❗️❗️❗️ **When you call `registerBlockType`, WordPress looks at the `name`.**
- When the content editor opens the block inserter and searches “testimonial”, it displays your block with the label "Testimonial Slider" from `title`.

---

## 🧠 Pro Tips for Block Developers

✅❗️❗️❗️ Use **clear and unique `name`** values. Conflicts across plugins or themes can cause block registration issues.

✅❗️❗️❗️ Make `title` **user-friendly and localized**. Use WordPress i18n functions like `__()` if you’re registering blocks manually.

✅❗️❗️❗️ Keep `name` static once released. If you must rename a block, consider creating a **new block** instead of renaming the existing one.

✅❗️❗️❗️ Use the `block.json` format to simplify registration. It allows WordPress to auto-discover and enqueue scripts, improving maintainability.

---

## 🚀 Final Thoughts

If you're building blocks for clients, users, or the WordPress community, remember:

- The `name` is for **WordPress and developers**.
- The `title` is for **content creators and users**.

Both are essential, and knowing when and how to use them makes your custom blocks robust, scalable, and user-friendly.

---

#WordPressDevelopment #GutenbergBlocks #ReactJS #blockjson #WPDev #JavaScript #CustomBlocks #WordPressTips #FullSiteEditing
