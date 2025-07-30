# Gutenberg Templates in WordPress – Deep Dive

## 🚀 What Are Gutenberg Templates?

**Gutenberg block templates** are pre-defined layouts made up of specific blocks. When a user adds a new post, page, or custom block (like your example), these templates automatically populate the editor with a default structure.

They are **not page templates or theme templates** — they’re **block structures inside the editor**.

---

## 🎯 Purpose of Templates

Templates help:

- 🛠 Guide the user with a fixed structure.
- ✅ Ensure content consistency.
- 🧱 Pre-fill content with default data (great for custom blocks).
- 🔐 Lock layout to prevent accidental changes.

---

## 🧠 Core Concepts You Should Understand

### 1. `template`

The `template` prop is an **array of arrays**, each representing a block and its attributes.

```js
template={[
  [ 'core/paragraph', { placeholder: 'Type here...' } ],
  [ 'core/image', {} ]
]}
```

### 2. `templateLock`

It locks the structure of the blocks inside a container:

- `'all'`: No blocks can be added, removed, or rearranged.
- `'insert'`: Users can reorder or edit existing blocks, but can’t insert new ones.
- `false` or `undefined`: Fully editable.

### 3. `allowedBlocks`

Limits the block types that can be inserted inside this container.

```js
<InnerBlocks allowedBlocks={["core/paragraph"]} />
```

---

## 🧪 Use Cases in Practice

### ✅ Use Case 1: Team Members Block (like your example)

Your block contains an `InnerBlocks` area that only allows `"create-block/team-member"`. The template defines **two default members** that show up when the block is first added.

```js
import { useBlockProps, InnerBlocks } from "@wordpress/block-editor";
import "./editor.scss";

export default function Edit() {
  return (
    <div {...useBlockProps()}>
      <InnerBlocks
        allowedBlocks={["create-block/team-member"]}
        template={[
          [
            "create-block/team-member",
            {
              name: "John Doe",
              position: "Developer",
            },
          ],
          [
            "create-block/team-member",
            {
              name: "Jane Smith",
              position: "Designer",
            },
          ],
        ]}
        templateLock="all"
      />
    </div>
  );
}
```

This:

- Ensures two team members always appear.
- Prevents users from adding/removing blocks (thanks to `templateLock="all"`).

---

### ✅ Use Case 2: Blog Post Layout Template

You can register templates for **custom post types** in `register_post_type()`:

```php
function myplugin_register_post_type() {
  register_post_type( 'portfolio', [
    'label' => 'Portfolio',
    'public' => true,
    'show_in_rest' => true,
    'template' => [
      [ 'core/image', [] ],
      [ 'core/heading', [ 'placeholder' => 'Project Title' ] ],
      [ 'core/paragraph', [ 'placeholder' => 'Project Description' ] ],
    ],
    'template_lock' => 'insert',
  ]);
}
add_action( 'init', 'myplugin_register_post_type' );
```

This sets a default layout for new Portfolio entries.

---

### ✅ Use Case 3: Reusable Container Layout

Imagine a “Pricing Table” block that always has **3 pricing tiers**:

```js
<InnerBlocks
  allowedBlocks={["myplugin/price-tier"]}
  template={[
    ["myplugin/price-tier", { title: "Basic", price: "$9" }],
    ["myplugin/price-tier", { title: "Pro", price: "$29" }],
    ["myplugin/price-tier", { title: "Enterprise", price: "$99" }],
  ]}
  templateLock="insert"
/>
```

---

## 💡 Tips & Best Practices

### ✅ Use `templateLock` carefully

Locking too tightly can frustrate users. `'insert'` is often a good middle ground.

### ✅ Combine `allowedBlocks` + `template`

This is powerful for building flexible yet structured content areas.

### ✅ Think UX

Templates improve editor UX by giving users a head-start instead of starting from a blank page.

### ✅ Use Templates for Onboarding

If you're building a custom block or plugin for clients or teams, templates reduce training time and ensure content uniformity.

---

## 📦 Bonus: Dynamic Templates with JavaScript

You can **generate templates dynamically** based on props or settings. Example:

```js
const teamMembers = ["Alice", "Bob", "Charlie"];
const template = teamMembers.map((name) => [
  "create-block/team-member",
  { name, position: "Member" },
]);
```

Pass `template` to `InnerBlocks`.

---

## 🔚 Final Thoughts

Templates are one of Gutenberg’s most **underrated features**. They’re a brilliant way to:

- Provide a ready-made structure,
- Guide users,
- Lock layouts for consistent design,
- And build polished editorial experiences inside the block editor.

---
