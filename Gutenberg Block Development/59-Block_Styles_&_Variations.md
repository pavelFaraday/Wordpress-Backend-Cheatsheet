# Block Styles & Block Variations in Gutenberg — Deep Dive + Interview Prep

## TL;DR

- **Block Styles** = alternate **looks** for an existing block. They only toggle a CSS class like `is-style-{name}` on the block wrapper (no attributes/structure change). Users switch them in the block’s **Styles** panel. ([WordPress Developer Resources][1])
- **Block Variations** = alternate **starting configurations** of a block (different **attributes and/or inner blocks**, own icon/title, can replace the default in the inserter). ([WordPress Developer Resources][2])

They solve different problems and can be used together (a variation can pre-apply a style by setting `className: "is-style-..."`). ([WordPress Developer Resources][2])

---

## What are Block Styles?

**Purpose.** Give authors one-click visual “skins” (e.g., Button: Fill vs Outline; Quote: Plain vs Fancy) without changing the block’s data. When selected, Gutenberg adds **`is-style-{name}`** to the block wrapper. One style can be marked default (no class added). ([WordPress Developer Resources][1])

**How they’re registered**

- **JS**: `wp.blocks.registerBlockStyle( 'core/quote', { name: 'fancy', label: 'Fancy' } )`. ([WordPress Developer Resources][1])
- **PHP**: `register_block_style( 'core/quote', [ 'name' => 'blue', 'label' => 'Blue Quote', … ] )`. Supports `inline_style`, `style_handle`, and since **WP 6.6**, **`style_data`** (theme.json-like array that integrates with Global Styles). ([WordPress Developer Resources][1])
- **Unregister**: JS `wp.blocks.unregisterBlockStyle()` or PHP `unregister_block_style()` (server-registered styles). Mind timing/race conditions when unregistering in JS. ([WordPress Developer Resources][1])

**Where styles live**

- CSS you write (via `inline_style` or enqueued file), **or** `style_data` (WP 6.6+) which lets the Style Engine output the CSS and exposes it to Global Styles. ([WordPress Developer Resources][1])

---

## What are Block Variations?

**Purpose.** Ship curated presets of a **single** block—different attributes and/or a starter **innerBlocks** layout—surfaced as separate items in the inserter, in transforms, or as default. Classic examples: **Embed** providers (YouTube, Twitter), **Media & Text** image-right default, **Query Loop** templates. ([WordPress Developer Resources][2])

**How they’re registered**

- **In the block definition**: add a `variations: [ … ]` array.
- **JS**: `wp.blocks.registerBlockVariation( 'core/media-text', { name, title, attributes, innerBlocks, scope, isDefault, isActive } )`. ([WordPress Developer Resources][2])
- **PHP**: filter `get_block_type_variations` to generate variations dynamically (e.g., by post types). ([WordPress Developer Resources][2])
- **Active detection**: use **`isActive`** (string\[] of attribute keys, or a function). **WP 6.6** improves specificity and nested paths (e.g., `'query.postType'`). ([WordPress Developer Resources][2])
- **Default behavior**: Setting **`isDefault: true`** can **replace** the block’s standard inserter item (mind conflicts with other defaults; unregister competing defaults first). ([WordPress Developer Resources][2])

---

## Styles vs. Variations — the crisp difference

| Aspect      | **Block Styles**                                   | **Block Variations**                                               |
| ----------- | -------------------------------------------------- | ------------------------------------------------------------------ |
| Changes     | **Visual only** (adds `.is-style-*`)               | **Attributes** and/or **innerBlocks** (structure/content defaults) |
| Where       | Toggled in **Styles** section of block sidebar     | Shown as separate **Inserter** items / transforms; can be default  |
| Data impact | No attribute change                                | Initializes (and identifies) block instance via attributes         |
| Good for    | Skins/skins-once-click (rounded, outline, striped) | Presets (hero Group, Query layout, provider choice)                |

Citations: styles add an `is-style-…` class and can have a default; variations set attributes/innerBlocks and may override default inserter item. ([WordPress Developer Resources][1])

> Don’t confuse **Block Styles/Variations** with **Theme “Style Variations”** (files in `/styles/*.json` that skin an entire theme/site). Those are global, not per-block. ([WordPress Developer Resources][3])

---

## Practical use cases

- **Block Styles**

  - Image: “Rounded”, “Polaroid”, “Framed”.
  - Quote: “Plain”, “Pullquote”, “Blue Quote”.
  - Button: “Fill”, “Outline”.
    Add/override via PHP or JS; in WP 6.6+ you can supply style definitions with `style_data` for Global Styles integration. ([WordPress Developer Resources][1])

- **Block Variations**

  - Group: “Hero (full-bleed)” variation initializes background/media/spacing.
  - Query Loop: “Grid” / “List” presets with different templates.
  - Media & Text: ship a right-image default (`isDefault: true`).
  - Embed: YouTube/Twitter etc. (provider attribute). ([WordPress Developer Resources][2])

---

## Quick cheat-sheet

- **Add a style (JS):** `wp.blocks.registerBlockStyle( 'core/quote', { name:'fancy', label:'Fancy' } )`. **Result:** toggles `.is-style-fancy`. ([WordPress Developer Resources][1])
- **Add a style (PHP):** `register_block_style( 'core/quote', [ 'name'=>'blue', 'label'=>__('Blue'), 'style_data'=> [ … ] ] )` (WP **6.6+**). ([WordPress Developer Resources][1])
- **Make a style default:** `isDefault: true` (no class output). ([WordPress Developer Resources][1])
- **Add a variation (JS):** `wp.blocks.registerBlockVariation( 'core/group', { name:'hero', title:'Hero', attributes:{ align:'full' }, innerBlocks:[…], isDefault:false } )`. ([WordPress Developer Resources][2])
- **Variation scopes:** `inserter`, `block`, `transform`. Use `isActive` so the editor shows the right label/icon for selected instances; nested paths & specificity improved in **6.6**. ([WordPress Developer Resources][2])
- **Unregister:** `wp.blocks.unregisterBlockStyle()` / `wp.blocks.unregisterBlockVariation()` (ensure correct load order). ([WordPress Developer Resources][1])
- **Don’t mix up** with theme **Global Style Variations** in `/styles/*.json` (site-wide skins). ([WordPress Developer Resources][3])

---

## Best practices & gotchas

- **Choose the right tool**: if you’re changing **look only**, use a **style**. If you’re changing **starting content/attributes**, use a **variation**. ([WordPress Developer Resources][2])
- **Name + defaults**: Be explicit; one default style per block is typical; one default variation can replace the inserter item (watch for conflicts). ([WordPress Developer Resources][1])
- **Performance**: Register styles server-side where possible; the engine only enqueues assets when needed. `style_data` (6.6+) plugs straight into the Style Engine/Global Styles. ([WordPress Developer Resources][1])
- **Curate the UI**: Hide styles/variations you don’t want; unregister in JS after `domReady` to avoid race conditions. ([WordPress Developer Resources][1])

---

## Interview-style talking points (with strong answers)

**Q1. Block Styles vs Block Variations — what’s the difference?**
**A.** Styles are **visual skins** that add an `is-style-*` class—no attributes or structure change. Variations are **presets** that set **attributes/innerBlocks** and can appear as separate **Inserter** items (and even become the default). Use styles for looks; variations for starting configurations. ([WordPress Developer Resources][1])

**Q2. How do authors pick styles and variations in the editor?**
**A.** Styles appear in the block’s **Styles** sidebar; choosing one toggles `is-style-*`. Variations appear in the **Inserter/transform** UI with their own icon/title. A variation can set `className` to pre-apply a style. ([WordPress Developer Resources][1])

**Q3. What changed in WordPress 6.6 relevant to styles/variations?**
**A.** You can register block styles with **`style_data`** (a theme.json-like array) so Global Styles can manage them; and **`isActive`** for variations supports nested paths and has improved specificity. ([WordPress Developer Resources][1])

**Q4. When would you make a variation the default? Any caveats?**
**A.** Use **`isDefault: true`** to replace the base inserter item (e.g., Media & Text with image right). If multiple defaults exist, the **first registered** wins—unregister competing defaults to ensure yours applies. ([WordPress Developer Resources][2])

**Q5. How do these relate to theme “Style Variations” in `/styles/*.json`?**
**A.** Those are **global theme skins** (site-wide), not per-block features. Block styles/variations operate at the **block** level. ([WordPress Developer Resources][3])

**Q6. Performance and maintainability tips?**
**A.** Prefer server-side style registration (`register_block_style`) so WordPress enqueues assets only when needed; use `style_data` for Style-Engine-managed output; curate/limit options to keep authors on-brand and the UI clean. ([WordPress Developer Resources][1])
