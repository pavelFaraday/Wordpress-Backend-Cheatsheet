# Block Styles & Block Variations in Gutenberg

## TL;DR

- **Block Styles** = alternate **looks** for an existing block. They only toggle a CSS class like `is-style-{name}` on the block wrapper (no attributes/structure change). Users switch them in the block’s **Styles** panel.
- **Block Variations** = alternate **starting configurations** of a block (different **attributes and/or inner blocks**, own icon/title, can replace the default in the inserter). [38. Block Variations](38-block_variations.md)

They solve different problems and can be used together (a variation can pre-apply a style by setting `className: "is-style-..."`).

### 🟦 Block Styles

- **What it is**: Just different “skins” (looks) for the **same block**.
- **How it works**: WordPress adds a class like `is-style-fancy` to the block wrapper, and your CSS decides how it looks.
- **What it does _not_ change**: The block’s attributes or structure — only design.

✅ **Example:**

- You insert a **Quote block**.
- In the sidebar **Styles** panel, you see:

  - **Default** (normal quote)
  - **Large** (big text, bolder look)

- When you switch, WordPress only adds/removes `is-style-large` class.

So **content stays the same** → only visuals change.

### 🟩 Block Variations

- **What it is**: Different **presets** of a block.
- **How it works**: A variation can change attributes (settings), inner blocks, icon, and title. It can even replace the default option in the inserter.
- **What it does**: Provides different “versions” of a block to start with.

✅ **Example:**

- The **Columns block** has variations:

  - **50/50** (two equal columns)
  - **70/30** (wide + narrow column)
  - **Three columns**

- Each variation inserts the block with different inner blocks already set up.

So **structure/content starting point changes**.

### 🔄 How They Can Work Together

- A **variation** can also pre-apply a **style**.

✅ **Example:**

- You make a **Button block variation** called “Rounded CTA”.

  - It sets `className: "is-style-rounded"`.
  - It also sets the text color to white and background to blue.

- Now when you pick that variation, you get the right structure **and** style right away.

### 📝 TL;DR (in even simpler words)

- **Block Styles** = change the **clothes** of a block (CSS only).
- **Block Variations** = change the **DNA** of a block (different setup/structure).
- You can use both → a variation can come with a style already applied.

;';'

---

## 🟦 Block Styles (just looks)

![Block Styles](<JS Libraries/Block_styles.png>)

**Purpose**

Give authors one-click visual “skins” (e.g., Button: Fill vs Outline; Quote: Plain vs Fancy) without changing the block’s data. When selected, Gutenberg adds **`is-style-{name}`** to the block wrapper. One style can be marked default (no class added).

**How they’re registered**

- **JS**: `wp.blocks.registerBlockStyle( 'core/quote', { name: 'fancy', label: 'Fancy' } )`.
- **PHP**: `register_block_style( 'core/quote', [ 'name' => 'blue', 'label' => 'Blue Quote', … ] )`.

* Only add/remove a CSS class → you decide the design in CSS.
* Best for: multiple visual looks of the **same block content**.

✅ **Example: Quote Block Style**

```js
// In your plugin or theme JS
import { registerBlockStyle } from "@wordpress/blocks";

registerBlockStyle("core/quote", {
  name: "fancy",
  label: "Fancy",
});
```

```css
/* In your stylesheet */
.is-style-fancy {
  border-left: 5px solid hotpink;
  font-style: italic;
}
```

👉 When the user selects **Fancy** in the sidebar, the block gets:

```html
<blockquote class="wp-block-quote is-style-fancy">
  <p>Hello world</p>
</blockquote>
```

Only the **look changes**.

### 📝 Super Short Summary

- **Block Style** = just a new outfit → CSS only.
- **Block Variation** = a different twin → new setup/structure.
- You can mix them → variation can wear a style.
  you see how it looks in a real custom block folder.

---

## 🟩 What are Block Variations?

![Block Variations](<JS Libraries/variations.png>)

Block variations are easily confused with block styles. With a block style, you can change how the block looks with CSS, and the style can be selected in the block’s settings sidebar in the editor. With a block variation, you change the block settings and create a variation with those presets.

- Imagine one block can come in **different flavors**.
- A **variation** is just a **preset** of that block.
- It changes **settings (attributes)** or gives you a ready **inner layout**.
- But it’s still the **same block** under the hood.

✅ **Examples you already know**:

- **Embed block** → YouTube, Twitter, Vimeo (all variations of `core/embed`).
- **Columns block** → 50/50, 70/30, three columns (variations of `core/columns`).
- **Query Loop block** → Grid, List, Large posts (variations of `core/query`).

### 🛠 How to Register Variations

### 1. In `block.json` (simplest way)

```json
{
  "apiVersion": 2,
  "name": "mytheme/hero",
  "title": "Hero Block",
  "category": "layout",
  "variations": [
    {
      "name": "image-left",
      "title": "Image Left",
      "attributes": {
        "mediaPosition": "left"
      }
    },
    {
      "name": "image-right",
      "title": "Image Right",
      "attributes": {
        "mediaPosition": "right"
      },
      "isDefault": true
    }
  ]
}
```

👉 Result: In the inserter, you see **Image Left** and **Image Right** as two “buttons” for the same block.

### 2. In JavaScript

```js
import { registerBlockVariation } from "@wordpress/blocks";

registerBlockVariation("core/media-text", {
  name: "image-right",
  title: "Image Right",
  attributes: { mediaPosition: "right" },
  isDefault: true,
});

registerBlockVariation("core/media-text", {
  name: "image-left",
  title: "Image Left",
  attributes: { mediaPosition: "left" },
});
```

### 3. With Inner Blocks

You can also provide **starter content**:

```js
registerBlockVariation("core/columns", {
  name: "two-column-text",
  title: "Two Column Text",
  innerBlocks: [
    ["core/column", {}, [["core/paragraph", { content: "Left text" }]]],
    ["core/column", {}, [["core/paragraph", { content: "Right text" }]]],
  ],
});
```

👉 When user picks it, the block is inserted **already filled with two paragraphs**.

### 4. Dynamic (in PHP)

If you want variations **based on post types**:

```php
add_filter( 'get_block_type_variations', function( $variations, $block_name ) {
    if ( 'core/query' === $block_name ) {
        $variations[] = [
            'name'  => 'my-custom-query',
            'title' => 'Products Loop',
            'attributes' => [
                'query' => [ 'postType' => 'product' ],
            ],
        ];
    }
    return $variations;
}, 10, 2 );
```

### 📝 Super Short Summary

- **Variations = presets** of one block (different looks/layouts to start with).
- Can be registered in **block.json**, **JS**, or **PHP**.
- **Attributes** = settings (e.g., align right).
- **innerBlocks** = starter layout (pre-filled content).
- **isDefault: true** = makes this variation the one shown by default.

---

## Styles vs. Variations - the crisp difference

| Aspect      | **Block Styles**                                   | **Block Variations**                                               |
| ----------- | -------------------------------------------------- | ------------------------------------------------------------------ |
| Changes     | **Visual only** (adds `.is-style-*`)               | **Attributes** and/or **innerBlocks** (structure/content defaults) |
| Where       | Toggled in **Styles** section of block sidebar     | Shown as separate **Inserter** items / transforms; can be default  |
| Data impact | No attribute change                                | Initializes (and identifies) block instance via attributes         |
| Good for    | Skins/skins-once-click (rounded, outline, striped) | Presets (hero Group, Query layout, provider choice)                |

> Don’t confuse **Block Styles/Variations** with **Theme “Style Variations”** (files in `/styles/*.json` that skin an entire theme/site). Those are global, not per-block.

---

## Practical use cases

- **Block Styles**

  - Image: “Rounded”, “Polaroid”, “Framed”.
  - Quote: “Plain”, “Pullquote”, “Blue Quote”.
  - Button: “Fill”, “Outline”.
    Add/override via PHP or JS; in WP 6.6+ you can supply style definitions with `style_data` for Global Styles integration.

- **Block Variations**

  - Group: “Hero (full-bleed)” variation initializes background/media/spacing.
  - Query Loop: “Grid” / “List” presets with different templates.
  - Media & Text: ship a right-image default (`isDefault: true`).
  - Embed: YouTube/Twitter etc. (provider attribute).

---

## Best practices & gotchas

- **Choose the right tool**: if you’re changing **look only**, use a **style**. If you’re changing **starting content/attributes**, use a **variation**.
- **Name + defaults**: Be explicit; one default style per block is typical; one default variation can replace the inserter item (watch for conflicts).
- **Performance**: Register styles server-side where possible; the engine only enqueues assets when needed. 
- **Curate the UI**: Hide styles/variations you don’t want; unregister in JS after `domReady` to avoid race conditions.

---

## Interview-style talking points (with strong answers)

**📌 Q1. Block Styles vs Block Variations — what’s the difference?**
**A.** Styles are **visual skins** that add an `is-style-*` class—no attributes or structure change. Variations are **presets** that set **attributes/innerBlocks** and can appear as separate **Inserter** items (and even become the default). Use styles for looks; variations for starting configurations.

**📌 Q2. How do authors pick styles and variations in the editor?**
- Styles appear in the block’s **Styles** sidebar; choosing one toggles `is-style-*`. 
- Variations appear in the **Inserter/transform** UI with their own icon/title. A variation can set `className` to pre-apply a style.

**📌 Q3. When would you make a variation the default? Any caveats?**
**A.** Use **`isDefault: true`** to replace the base inserter item (e.g., Media & Text with image right). If multiple defaults exist, the **first registered** wins.

**📌 Q4. How do these relate to theme “Style Variations” in `/styles/*.json`?**
**A.** Those are **global theme skins** (site-wide), not per-block features. Block styles/variations operate at the **block** level.

**Q6. Performance and maintainability tips?**
**A.** Prefer server-side style registration (`register_block_style`) so WordPress enqueues assets only when needed; use `style_data` for Style-Engine-managed output; curate/limit options to keep authors on-brand and the UI clean.
