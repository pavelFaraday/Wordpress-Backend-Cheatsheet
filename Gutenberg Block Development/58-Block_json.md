# `block.json` in Gutenberg

## 🗂 What is `block.json`?

- It’s a **small JSON file** that describes your block.
- Think of it as the **ID card** (name, title, icon, category, etc.), **state** (attributes) + **settings** (styles, scripts, supports).
- WordPress reads it in **PHP** and **JS**, so **both sides share the same truth**.

### 🎯 Why does it exist?

Before, you had to register blocks **twice**:

- In **PHP** with `register_block_type()`
- In **JS** with `registerBlockType()`

This often got out of sync.

👉 `block.json` fixes that by being the **one file everyone trusts**.

- PHP reads it with `register_block_type_from_metadata()`
- JS can `import metadata from './block.json'`

So you only define things **once**.

### ✅ Example

`block.json` for a simple “Testimonial” block:

```json
{
  "apiVersion": 2,
  "name": "myplugin/testimonial",
  "title": "Testimonial",
  "icon": "format-quote",
  "category": "widgets",
  "description": "A customer testimonial block.",
  "style": "file:./style.css",
  "editorScript": "file:./index.js",
  "supports": {
    "html": false,
    "align": true
  }
}
```

### What happens:

- WordPress automatically loads `style.css` on front + editor.
- Loads `index.js` in editor.
- Shows the block in inserter with a quote icon.
- Both PHP and JS use the same info → no duplication.

### 📝 Super Short Summary

- **`block.json` = block’s blueprint.**
- Keeps PHP + JS **in sync**.
- Autoloads scripts/styles.
- Makes blocks easier to share, discover, and maintain.

---

## Core concepts (how it all fits)

- **Identity**: `name` (immutable, used in post content), `title` (human label), `category`, `icon`, `keywords`.
- **Capabilities**: `supports` toggles features the block exposes (color, spacing, typography, alignment, HTML editing, etc.).
- **Data model**: `attributes` define what the block saves/reads (and how).
- **Context**: `providesContext` (what this block exposes to descendants) and `usesContext` (what it consumes from ancestors) for parent→child data flows.
- **Assets**: `editorScript`, `script`, `viewScript`, `editorStyle`, `style` (+ **file:** shorthands) declare loadable assets; PHP auto-registers/queues them.
- **Behavior shape**: **Where and how do I appear?**
  `apiVersion`: block API version (2 is current).
  `parent / ancestor`: restrict where block can be inserted.
  `styles`: block style variations (like Default / Fancy).
  `variations`: curated presets (like Columns 50/50, 70/30).
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

## 🌟 Why `block.json` is the “heart” of modern blocks

### 1. **One file, both worlds**

- Instead of registering a block in **PHP** and again in **JS**, you now write everything once.
- WordPress reads it in PHP (to register, load assets) **and** in JS (to build editor behavior).
- ✅ Less duplication, fewer mistakes.

### 2. **Good for tooling**

- Build tools, scaffolding commands (`wp-scripts`, `@wordpress/create-block`), and even IDEs can read the same file.
- ✅ Developers, designers, and automation tools all understand your block just by looking at `block.json`.

### 3. **Better performance**

- You don’t need to enqueue scripts or styles manually with `wp_enqueue_*`.
- WordPress only loads the assets you declare in `block.json` — no extra files, no wasted bandwidth.

### 4. **Clear collaboration**

- Designers and product managers can open `block.json` and immediately see:

  - Features the block supports (colors, spacing, typography).
  - Available styles and variations.
  - Which assets it uses.

- ✅ Everyone speaks the same language, not just devs.

### 5. **Future-proof**

- As WordPress adds new features (like new `supports`, new `context` options), you just declare them in `block.json`.
- ✅ You don’t rewrite code — you just update metadata.

### 📝 Super Short Summary

`block.json` is central because it’s:

- **One truth** for PHP + JS.
- **Readable by humans and tools**.
- **Optimized** (loads only what’s needed).
- **Collaborative** (product, design, devs see the same info).
- **Future-ready** (easy to adopt new features).

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

# Starter `block.json` Template Pack

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

> In `edit()`, read the context with 
`useSelect( select => select( 'core/block-editor' ).getBlockParentsByBlockName( … ) )` 
or via props from the editor (your `edit` receives `context`).

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
