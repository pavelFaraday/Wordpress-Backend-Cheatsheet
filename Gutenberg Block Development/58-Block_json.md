# **block.json in Gutenberg — Deep Dive + Interview Prep**

## What `block.json` is (and why it exists)

`block.json` is the **single source of truth** for a block’s metadata: the block’s identity, capabilities, editor/front-end assets, default styles, variations, and how it interacts with the editor. It replaces scattered PHP/JS registration by **one declarative file** that:

- Registers the block on the **PHP** side (via `register_block_type_from_metadata()`), autoloading scripts/styles listed in the file.
- Feeds the **JS** side (`registerBlockType`) with the same data (you can `import metadata from './block.json'` and reuse it).
- Keeps editor and front-end **in sync** (assets, features, supports), and makes blocks **discoverable** to tooling.

> Bottom line: in the WordPress 6+ era, **block.json is the canonical contract** for your block.

---

## Core concepts (how it all fits)

- **Identity**: `name` (immutable, used in post content), `title` (human label), `category`, `icon`, `keywords`.
- **Capabilities**: `supports` toggles features the block exposes (color, spacing, typography, alignment, HTML editing, etc.).
- **Data model**: `attributes` define what the block saves/reads (and how).
- **Context**: `providesContext` (what this block exposes to descendants) and `usesContext` (what it consumes from ancestors) for parent→child data flows.
- **Assets**: `editorScript`, `script`, `viewScript`, `editorStyle`, `style` (+ **file:** shorthands) declare loadable assets; PHP auto-registers/queues them.
- **Behavior shape**: `apiVersion`, `parent`/`ancestor` (where it can be inserted), `styles` (style variations), `variations` (block presets), `example` (inserter preview).
- **Dynamic rendering**: `render` can point to a PHP file for server-side output.

---

## The requested distinctions (clear & concise)

### `title` vs `name` vs `category` vs `supports` vs `attributes` vs `providesContext` vs `usesContext`

- **`name`**: Machine identifier **`namespace/block-slug`**. It’s **written into post content** (`<!-- wp:ns/block {...} /-->`). **Do not change** after release.
- **`title`**: Human-readable label shown in the inserter (localizable). Safe to change/translate anytime.
- **`category`**: Inserter grouping (“text”, “media”, “design”, “widgets”, “theme”, “embed”, or a custom category you register). Doesn’t affect behavior—only **where users find it**.
- **`supports`**: On/off switches for editor features the block **can** expose (e.g., color controls, spacing, typography, alignment, anchor, multiple/inserter toggles, custom class, HTML editing).
  _Think: “what the UI allows on this block.”_
- **`attributes`**: Schema describing the block’s **saved data**—its properties, types, defaults, and where they’re sourced from (e.g., HTML, text, meta, query).
  _Think: “what the block stores/reads.”_
- **`providesContext`**: Declares **values this block provides** to its descendants (e.g., a “level” or “color” that children can inherit). Maps a **context key** to this block’s value (typically an attribute path).
- **`usesContext`**: Declares **which context keys this block consumes** from ancestors. The editor injects those values; your block reads them (usually in `edit()`/`save()`).

---

## Why `block.json` is central in modern block registration

- **Single declaration** drives both **PHP** (registration + asset loading) and **JS** (editor behavior), minimizing drift.
- **Tooling-friendly**: scaffolding, build tools, and dependency resolution use the same metadata.
- **Performance & correctness**: WordPress loads only the assets you state; no manual `wp_enqueue_*` guesswork.
- **Easier collaboration**: Designers/devs see capabilities, styles, and tokens in **one file**; product can review features at a glance.
- **Forward compatibility**: New capabilities (e.g., new supports, context keys) can be adopted declaratively.

---

## Practical, high-value fields (what you’ll set most)

**Identity & UI**

- `apiVersion`: usually `2` for modern blocks.
- `name`, `title`, `description`, `icon`, `keywords`, `category`.

**Capabilities**

- `supports`:

  - **Color**: `{"color": {"background": true, "text": true, "link": true}}`
  - **Spacing**: `{"spacing": {"margin": true, "padding": true, "blockGap": true}}`
  - **Typography**: `{"typography": {"fontSize": true, "lineHeight": true, "fontFamily": true}}`
  - **Layout/alignment**: `{"align": ["wide", "full"]}`, `{"anchor": true}`
  - **UX flags**: `{"html": false, "multiple": false, "inserter": true, "className": true, "customClassName": true}`

- _Tip_: Theme authors can **further restrict** these via `theme.json` (the block must support a feature for the theme to expose it).

**Data Model**

- `attributes`: `"type"`, `"default"`, `"source"` (e.g., `"attribute"`, `"text"`, `"html"`, `"meta"`, `"query"`), and `"selector"`/`"attribute"` for DOM extraction.

**Context**

- `providesContext`: e.g., `"my/ns:level": "attributes.level"`.
- `usesContext`: e.g., `["my/ns:level"]`.

**Insertion rules**

- `parent`: array of allowed direct parents (restricts insertion).
- `ancestor`: stricter ancestry constraint (anywhere within those blocks).

**Assets**

- `editorScript`, `script`, `viewScript`, `editorStyle`, `style`

  - Accept **handles** or **`file:./relative-path.js`** shorthands (PHP will register + resolve dependencies using generated `*.asset.php` files).

**Presentation & presets**

- `styles`: block style variations (e.g., “rounded”, “outline”).
- `variations`: named presets of attributes/inner blocks; can mark one as `isDefault`.
- `example`: preview used in the Inserter.

**Dynamic output**

- `render`: `file:./render.php` (server-side rendered block).

---

## How it plays with `theme.json` (quick mental model)

- **Block “supports”** (in `block.json`) say what a block **can** do.
- **Theme “settings/styles”** (in `theme.json`) decide what the **user is allowed** to do globally and which **design tokens** they can pick.
- The editor shows **the intersection** of block supports × theme settings.

---

## Quick cheat-sheet (keep nearby)

**Immutable**: `name`
**Safe to change**: `title`, `description`, `keywords`, `icon`, `category`

**Most used**

- `supports.color/spacing/typography/align/anchor/html/multiple/inserter/className`
- `attributes` (type, default, source, selector)
- `editorScript` / `script` / `viewScript` / `editorStyle` / `style` (handles **or** `file:`)
- `variations`, `styles`, `example`
- `parent` / `ancestor`
- `providesContext` / `usesContext`
- `render` (dynamic blocks)

**Rules of thumb**

- Pick `name` carefully — it’s forever.
- Start with `apiVersion: 2`.
- Only enable `supports` you truly need; let `theme.json` refine UI globally.
- Prefer `file:` shorthands for assets; let WP handle dependencies.
- Use `variations` for UX (pre-configured flavors); use `styles` for visual skins.

---

## Use cases you’ll actually build

- **Marketing card block** with:

  - `supports.color/spacing/typography` for on-brand flexibility
  - `attributes` for title, image URL, CTA text
  - `variations` for “Hero”, “Compact”, “With Icon”

- **Section block** that **providesContext** for “tone” or “level”; child blocks `usesContext` to adjust their defaults.
- **Dynamic listing** block with `render` (PHP) to query posts and print markup; `viewScript` hydrate/interact on the front-end.
- **Locked layout** using `parent`/`ancestor` so inner blocks can only exist where intended.

---

## Interview-style talking points (with strong answers)

**Q1. Why is `block.json` the center of modern block development?**
**A.** It’s a **single, declarative contract** that powers both server and client registration. PHP auto-loads registered assets and render callbacks; JS consumes the same metadata to register behavior. This eliminates duplication, prevents drift, and makes blocks easy to discover, build, and maintain at scale.

**Q2. Explain the difference between `name`, `title`, and `category`.**
**A.** `name` is the **immutable** machine ID (`ns/slug`) stored in content; `title` is the human label (safe to change/translate); `category` just controls inserter grouping/organization.

**Q3. How do `supports` and `attributes` differ?**
**A.** `supports` toggles **which editor controls** are exposed (color, spacing, typography, etc.). `attributes` define the **block’s data model**—the properties it saves/reads and how they’re sourced from the DOM or meta.

**Q4. What are `providesContext` and `usesContext`, and when would you use them?**
**A.** They’re the block editor’s context system. `providesContext` exposes values (e.g., a “level” or “theme”) to descendants; `usesContext` reads those values. Use it to create **hierarchical blocks** where parent settings cascade (section → headings/buttons).

**Q5. How do you restrict where a block can appear?**
**A.** Use `parent` for direct parent restriction and `ancestor` for broader ancestry rules. This ensures blocks only appear within allowed structures (e.g., “Only inside Columns”).

**Q6. How do `block.json` and `theme.json` interact?**
**A.** `block.json` defines what the block **can** do (`supports`); `theme.json` defines what the **site allows** globally (tokens and permitted controls). The UI shows the intersection, keeping authors on-brand.

**Q7. How do you ship multiple looks or presets of a block?**
**A.** Use `styles` for **style variations** (visual skins), and `variations` to pre-fill **attributes/inner blocks** for distinct starting points in the Inserter. You can mark one variation as `isDefault`.

**Q8. When do you choose a dynamic (server-rendered) block?**
**A.** When output depends on runtime data (queries, user state, time). Set `render` to a PHP file; pair with `viewScript` for hydration/interaction as needed.

**Q9. What are common pitfalls with `block.json`?**
**A.** Changing `name` after release (breaks existing content); enabling too many `supports` (bloated UI); forgetting asset registration (not using `file:` therefore no deps); over-relying on attributes when context or variations would be cleaner.

**Q10. How do you keep blocks forward-compatible?**
**A.** Keep `name` stable; evolve `attributes` with **deprecations** (in JS) to migrate saved content; gate features behind `supports`; let `theme.json` handle global control visibility and tokens.

---

# Starter `block.json` Template Pack (Gutenberg, WP 6+)

Copy–paste, then tweak values (`namespace`, slugs, titles, assets). All examples use **`apiVersion: 2`** and are compatible with WP 6+.

> ✅ **How to load**: Put each `block.json` in its block folder (e.g., `blocks/hero/block.json`). In your plugin/theme bootstrap, call `register_block_type_from_metadata( __DIR__ . '/blocks/hero' );` (PHP). WordPress will auto‑register and enqueue assets referenced via `file:…`.

## 1) Minimal **Static Block** (safe default)

```json
{
  "apiVersion": 2,
  "name": "ns/static-card",
  "title": "Static Card",
  "description": "A simple content card with heading and text.",
  "category": "design",
  "icon": "index-card",
  "textdomain": "my-text-domain",

  "keywords": ["card", "box", "panel"],
  "supports": {
    "html": false,
    "anchor": true,
    "className": true,
    "customClassName": true,
    "color": { "background": true, "text": true, "link": true },
    "spacing": { "margin": true, "padding": true, "blockGap": true },
    "typography": { "fontSize": true, "lineHeight": true, "fontFamily": true },
    "align": ["wide", "full"]
  },

  "attributes": {
    "title": {
      "type": "string",
      "source": "html",
      "selector": "h3",
      "default": "Card Title"
    },
    "content": {
      "type": "string",
      "source": "html",
      "selector": "p",
      "default": "Card text…"
    }
  },

  "editorScript": "file:./index.js",
  "editorStyle": "file:./editor.css",
  "style": "file:./style.css",

  "example": {
    "attributes": { "title": "Hello", "content": "Preview example text." }
  }
}
```

## 2) **Dynamic (Server‑Rendered) Block** (PHP `render`)

```json
{
  "apiVersion": 2,
  "name": "ns/dynamic-list",
  "title": "Dynamic List",
  "description": "Server‑rendered list (e.g., recent posts).",
  "category": "widgets",
  "icon": "list-view",
  "textdomain": "my-text-domain",

  "supports": {
    "html": false,
    "anchor": true,
    "className": true,
    "spacing": { "margin": true, "padding": true },
    "color": { "text": true }
  },

  "attributes": {
    "postsToShow": { "type": "number", "default": 5 },
    "showDate": { "type": "boolean", "default": true }
  },

  "render": "file:./render.php",
  "editorScript": "file:./index.js",
  "style": "file:./style.css",
  "viewScript": "file:./view.js",

  "example": {
    "attributes": { "postsToShow": 3, "showDate": true }
  }
}
```

> Put your SSR markup in `render.php`. Use `view.js` for front‑end interactivity (hydration, toggles, etc.).

## 3) **Parent → Child Context** (provides/uses)

### Parent provides a value (e.g., visual **tone**)

```json
{
  "apiVersion": 2,
  "name": "ns/section",
  "title": "Section",
  "category": "design",
  "icon": "editor-table",

  "attributes": {
    "tone": { "type": "string", "default": "brand" }
  },

  "providesContext": {
    "ns:tone": "attributes.tone"
  },

  "supports": {
    "html": false,
    "color": { "background": true },
    "spacing": { "padding": true, "margin": true }
  },

  "editorScript": "file:./index.js",
  "style": "file:./style.css"
}
```

### Child consumes the parent context

```json
{
  "apiVersion": 2,
  "name": "ns/section-heading",
  "title": "Section Heading",
  "category": "text",
  "icon": "heading",

  "usesContext": ["ns:tone"],

  "attributes": {
    "content": {
      "type": "string",
      "source": "html",
      "selector": "h2",
      "default": "Section title"
    }
  },

  "supports": {
    "html": false,
    "typography": { "fontSize": true, "fontFamily": true },
    "color": { "text": true }
  },

  "parent": ["ns/section"],
  "editorScript": "file:./index.js",
  "style": "file:./style.css"
}
```

> In `edit()`, read the context with `useSelect( select => select( 'core/block-editor' ).getBlockParentsByBlockName( … ) )` or via props from the editor (your `edit` receives `context`).

## 4) **Variations** (pre‑configured presets in the inserter)

```json
{
  "apiVersion": 2,
  "name": "ns/cta",
  "title": "CTA",
  "category": "design",
  "icon": "megaphone",

  "attributes": {
    "title": {
      "type": "string",
      "source": "html",
      "selector": "h3",
      "default": "Call to Action"
    },
    "url": { "type": "string", "default": "#" }
  },

  "variations": [
    {
      "name": "primary",
      "title": "Primary CTA",
      "isDefault": true,
      "attributes": { "title": "Get Started", "url": "/start" }
    },
    {
      "name": "secondary",
      "title": "Secondary CTA",
      "attributes": { "title": "Learn More", "url": "/learn" }
    }
  ],

  "supports": {
    "html": false,
    "spacing": { "padding": true },
    "color": { "background": true, "text": true }
  },
  "editorScript": "file:./index.js",
  "style": "file:./style.css"
}
```

## 5) **Style Variations** (visual skins for a single block)

```json
{
  "apiVersion": 2,
  "name": "ns/notice",
  "title": "Notice",
  "category": "design",
  "icon": "warning",

  "supports": {
    "html": false,
    "color": { "background": true, "text": true },
    "spacing": { "padding": true }
  },

  "styles": [
    { "name": "info", "label": "Info", "isDefault": true },
    { "name": "success", "label": "Success" },
    { "name": "warning", "label": "Warning" },
    { "name": "danger", "label": "Danger" }
  ],

  "attributes": {
    "content": {
      "type": "string",
      "source": "html",
      "selector": "p",
      "default": "Heads up!"
    }
  },

  "editorScript": "file:./index.js",
  "style": "file:./style.css"
}
```

> Provide CSS for `.is-style-info`, `.is-style-success`, etc., in `style.css`.

## 6) **Locked Placement** (parent/ancestor constraints)

```json
{
  "apiVersion": 2,
  "name": "ns/column-card",
  "title": "Column Card",
  "category": "design",
  "icon": "columns",

  "parent": ["core/column"],
  "ancestor": ["core/columns"],

  "supports": { "html": false, "spacing": { "margin": true, "padding": true } },
  "attributes": { "title": { "type": "string", "default": "Card" } },

  "editorScript": "file:./index.js",
  "style": "file:./style.css"
}
```

## 7) **Media‑rich Block** (with `viewScript` only)

```json
{
  "apiVersion": 2,
  "name": "ns/lightbox-image",
  "title": "Lightbox Image",
  "category": "media",
  "icon": "format-image",

  "attributes": {
    "url": { "type": "string" },
    "alt": { "type": "string", "default": "" },
    "caption": { "type": "string", "source": "html", "selector": "figcaption" }
  },

  "supports": { "html": false, "anchor": true, "color": { "text": true } },

  "editorScript": "file:./index.js",
  "style": "file:./style.css",
  "viewScript": "file:./view.js",

  "example": {
    "attributes": { "url": "/images/example.jpg", "caption": "A scenic view" }
  }
}
```

## Notes & Best Practices

- **Name is forever**: choose `namespace/slug` carefully; it’s stored in post content.
- **Enable only needed supports**: keep the UI focused; let `theme.json` expose tokens & refine controls.
- **Use `file:` shorthands** for assets; WP resolves handles and dependencies via generated `*.asset.php`.
- **Preview matters**: set a realistic `example` for a better Inserter experience.
- **Context for composition**: parent blocks should `provideContext`; children `useContext` for clean cascading behavior.
- **Dynamic blocks**: prefer `render` when output depends on runtime (queries, permissions, time). Pair with `viewScript` for interactivity.

## Handy PHP bootstrap (drop‑in)

```php
<?php
/** Plugin Name: My Blocks */
add_action( 'init', function() {
  // Register multiple blocks from their directories
  foreach ( [ 'static-card', 'dynamic-list', 'section', 'section-heading', 'cta', 'notice', 'column-card', 'lightbox-image' ] as $block ) {
    register_block_type_from_metadata( __DIR__ . "/blocks/$block" );
  }
});
```

## Optional: Project Structure (suggested)

```
my-plugin/
└─ blocks/
   ├─ static-card/
   │  ├─ block.json
   │  ├─ index.js
   │  ├─ editor.css
   │  └─ style.css
   ├─ dynamic-list/
   │  ├─ block.json
   │  ├─ index.js
   │  ├─ render.php
   │  ├─ view.js
   │  └─ style.css
   ├─ section/
   ├─ section-heading/
   ├─ cta/
   ├─ notice/
   ├─ column-card/
   └─ lightbox-image/
```

---
