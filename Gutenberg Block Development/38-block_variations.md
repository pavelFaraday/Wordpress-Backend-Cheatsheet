# 🔍 What Are Block Variations in Gutenberg?

**Block Variations** in Gutenberg are a way to create multiple preset versions of a block—especially **InnerBlocks-based blocks** (like Group, Columns, or your custom blocks). They allow you to:

* Preconfigure default settings or attributes
* Predefine inner block layouts
* Give custom titles/icons/keywords to distinguish them in the block inserter

✅ In short, **block variations are like templates for blocks.**

---

## 🎯 Purpose of Variations

Block Variations aim to:

1. **Simplify user experience**
   Let users pick from ready-made configurations rather than building from scratch every time.

2. **Enable design consistency**
   You can define standard layouts (e.g. 2-column CTA, testimonials section) across your site or for clients.

3. **Extend reusable blocks**
   Instead of duplicating a block with slight changes, use variations to extend and adapt a single base block.

---

## ⚙️ Core Concepts

Here are the main parts involved:

### 1. `block.json`

The variations can be **defined statically** in `block.json` (Gutenberg 9.1+):

```json
{
  "name": "myplugin/my-block",
  "title": "My Block",
  ...
  "variations": [
    {
      "name": "hero",
      "title": "Hero Block",
      "description": "A variation with a heading and image",
      "attributes": {
        "alignment": "full"
      },
      "innerBlocks": [
        ["core/heading", { "level": 1 }],
        ["core/image"]
      ],
      "icon": "format-image",
      "isDefault": true
    }
  ]
}
```

### 2. `registerBlockVariation` (Dynamic registration via JS)

In cases where you want **more control** or dynamic behavior (e.g., user permissions, locales), use:

```js
import { registerBlockVariation } from '@wordpress/blocks';

registerBlockVariation( 'core/group', {
  name: 'cta-section',
  title: 'CTA Section',
  icon: 'megaphone',
  description: 'Call to action layout with text and button.',
  attributes: {
    className: 'cta-layout',
  },
  innerBlocks: [
    [ 'core/paragraph', { placeholder: 'Add your text...' } ],
    [ 'core/button' ],
  ],
  scope: [ 'inserter' ],
} );
```

---

## 🚀 Use Cases in Practice

Here are some **practical examples** of how variations are used in Gutenberg development:

---

### 1. **Custom Hero Layouts for Landing Pages**

You might register a variation of the Group block:

```js
registerBlockVariation('core/group', {
  name: 'hero-section',
  title: 'Hero Layout',
  attributes: {
    className: 'hero-section'
  },
  innerBlocks: [
    ['core/heading', { level: 1 }],
    ['core/paragraph'],
    ['core/button', { text: 'Get Started' }]
  ],
});
```

✅ Useful for clients who want multiple pre-styled hero banners.

---

### 2. **Team Member Cards**

For a custom `team-member` block, you can define variations for different layouts:

* Layout 1: Image on top
* Layout 2: Image left, content right

```js
registerBlockVariation('myplugin/team-member', {
  name: 'with-image-top',
  title: 'Image on Top',
  attributes: { layout: 'top' }
});
```

✅ Makes it easy to switch between designs visually without modifying code.

---

### 3. **Testimonials Section**

You can create variations for:

* Single testimonial
* Grid of testimonials
* Carousel (with block style or variation control)

Each with default InnerBlocks.

---

## 🛠 Best Practices

1. **Use variations for reusable layouts**
   If your block requires repeated layouts, this is more efficient than duplicating.

2. **Prefer `block.json` when static**
   For better performance and compatibility.

3. **Use `registerBlockVariation` when dynamic**
   Like loading variations based on user role, locale, or custom logic.

4. **Give visual clues (icon, title)**
   Helps users quickly understand what each variation is.

5. **Scope control**
   Use the `scope` option to control where the variation appears:

   ```js
   scope: [ 'inserter', 'transform' ] // or just 'transform'
   ```

---

## 🧩 Differences: Block Styles vs Variations

| Feature               | Block Styles | Block Variations            |
| --------------------- | ------------ | --------------------------- |
| Controls Design?      | ✅ Yes        | ✅ Sometimes (via className) |
| Controls Layout?      | ❌ No         | ✅ Yes                       |
| Appears in Inserter?  | ❌ No         | ✅ Yes                       |
| Supports InnerBlocks? | ❌ No         | ✅ Yes                       |

Use **Styles** for theme-level design, **Variations** for structure/layout/content presets.

---

## ✅ Summary

✔️ **Block Variations = Powerful customization tool** for Gutenberg
✔️ Ideal for **layout presets, reusable patterns, design systems**
✔️ Can be **static (block.json)** or **dynamic (JS API)**
✔️ Supports **InnerBlocks**, **scoping**, **custom attributes**, **icons**, and **transforms**

---

## 📦 Extra: Variation Transforms

You can allow **transforming between variations** like this:

```js
registerBlockVariation( 'core/group', {
  name: 'two-columns',
  title: 'Two Columns',
  isActive: ( attributes, variationAttributes ) => {
    return attributes.className === 'two-col-layout';
  },
  transforms: {
    from: [
      {
        type: 'block',
        blocks: [ 'core/group' ],
        transform: ( attributes ) => {
          return createBlock( 'core/group', {
            ...attributes,
            className: 'two-col-layout',
          });
        },
      },
    ],
  },
});
```
