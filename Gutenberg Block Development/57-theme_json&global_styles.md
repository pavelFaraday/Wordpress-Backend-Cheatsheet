# theme.json & Global Styles in Gutenberg

## What they are (in one line)

### 📌 `theme.json` & Global Styles — Super Short Summary

- **What `theme.json` is:** A JSON file in a block theme = **single source of truth** for design tokens (colors, typography, spacing) and editor capabilities (what controls users can access).

- **What Global Styles is:** The **Site Editor UI** that surfaces these settings, lets users override them, and stores changes in the database.

- **Key roles of `theme.json`:**

  - **Settings:** Enable/disable editor features (`color`, `spacing`, `typography`, etc.).
  - **Styles:** Define default styles for elements (e.g., `body`, `h1`) and blocks (e.g., `core/button`).
  - **Presets:** Provide palettes, font sizes, gradients.
  - **Structure:** Hierarchy = global → elements → blocks.
  - **Performance:** Generates optimized CSS for front-end & editor automatically.

- **How Global Styles fits:**

  - Reads `theme.json` → displays controls in the editor.
  - User customizations are saved in DB and merged on render.
  - Ensures themes stay **consistent, customizable, and future-proof**.

> 👉 In short: **`theme.json` defines the design system + editor features; Global Styles is the UI that applies and customizes them.**

---

## 📌 Styles Flow — Super Short Summary

1️⃣ **Block supports** → Each block declares what style options it can use (color, typography, spacing, etc.).
2️⃣ **theme.json (settings)** → Theme enables/disables those features + defines design tokens (palette, fonts, spacing scale).
3️⃣ **CSS variables** → WordPress auto-generates `--wp--preset--*` vars + utility classes (`.has-*`).
4️⃣ **Editor UI** → Users see palettes, font pickers, spacing controls (from block + theme.json).
5️⃣ **Global Styles (user overrides)** → Site Editor saves custom styles in DB, overriding theme defaults.
6️⃣ **Final styles** → WP Style Engine outputs optimized CSS → applied consistently in editor + frontend.

👉 In short: **Blocks declare features → theme.json defines tokens → WP generates CSS vars → editor shows controls → users override via Global Styles → Style Engine builds final CSS.**

👉 In short:

- **`settings` = the rules + tokens (what user can do, what variables exist).**
- **`styles` = the actual CSS that gets applied globally or per block.**

---

## 📌 Style Engine & Global Styles

### 🔧 Style Engine

- Converts **theme.json + user overrides** → CSS vars, utility classes, compiled CSS.
- Ensures **same output** for **editor + frontend**.
- Uses helpers like `wp_style_engine_get_styles()` (core calls this internally).
- Handles tokens → `--wp--preset--*` vars, `.has-*` classes, optimized inline CSS.

### 🎨 Global Styles (UI)

- Site Editor panel where users adjust **colors, typography, spacing, etc.**
- Choices are saved in DB as **user-level styles**.
- Merge order: **Core → Theme → Child Theme → User** (user wins).
- Lets non-developers override theme defaults while staying in design system.

👉 In short: **Style Engine builds the CSS; Global Styles is the UI where users customize it, stored in DB, overriding theme.json defaults.**

---

## 📌 Style Variations

- **What they are:** Extra JSON files in `/styles/*.json` → alternate “skins” for a block theme.
- **Purpose:** Ship multiple design flavors (e.g., light/dark, brand palettes, typography sets).
- **How it works:**

  - Each variation = full or partial `theme.json`.
  - User selects in Site Editor → data saved as a **user customization** (DB).
  - ⚠️ Once customized, future updates to that variation won’t auto-apply.

- **Use cases:** Light/dark mode, corporate brand variations, seasonal design tweaks.
- **Benefit:** Productize themes → multiple looks with one codebase.

👉 In short: **Style Variations are theme.json “skins” shipped in `/styles/`, giving users ready-made design options (like light/dark) without new themes.**

---

# 1. 🧩 How Block Supports Interact with `theme.json`

- **Block supports** = a block’s ability to handle styles (color, typography, spacing, etc.).
- They’re declared by **block authors** in `block.json` or internally in core blocks. Example: `"supports": { "color": { "background": true } }`.
- **theme.json bridges this**:

  - If a block declares support for colors, and your `theme.json` defines a color palette, then the block’s UI shows that palette.
  - If you disable something in `theme.json` (e.g., `"color": { "custom": false }`), the editor **hides** the custom color picker even if the block supports it.

- **Result**: `theme.json` doesn’t invent features — it configures and restricts what block supports expose to the user.

👉 Think of block supports as **“what a block can do”**, and `theme.json` as **“what the theme lets the user do.”**

# 2. 🎨 Defining Global Design Tokens (colors, spacing, typography)

`theme.json` lets you define **presets** that become **CSS variables** and **utility classes**:

### a) Colors

```json
"settings": {
  "color": {
    "palette": [
      { "slug": "brand", "color": "#0055ff", "name": "Brand Blue" },
      { "slug": "accent", "color": "#ff6600", "name": "Accent Orange" }
    ],
    "custom": false
  }
}
```

- Produces CSS vars like `--wp--preset--color--brand`.
- Adds classes like `.has-brand-color`.
- Blocks with color support (e.g. Button, Paragraph) show this palette.

### b) Typography

```json
"settings": {
  "typography": {
    "fontFamilies": [
      { "slug": "heading", "fontFamily": "Georgia, serif", "name": "Heading" },
      { "slug": "body", "fontFamily": "Arial, sans-serif", "name": "Body" }
    ],
    "fluid": true,
    "customFontSize": false
  }
}
```

- Creates variables like `--wp--preset--font-family--heading`.
- Adds classes like `.has-heading-font-family`.
- `fluid: true` enables **fluid typography** (font sizes scale with viewport).

---

### c) Spacing

```json
"settings": {
  "spacing": {
    "spacingScale": {
      "steps": 5,
      "mediumStep": 1.125
    },
    "blockGap": true,
    "units": ["px","em","rem"]
  }
}
```

- Generates a spacing scale (small → large).
- `blockGap` controls space between blocks.
- Blocks with margin/padding support inherit these tokens.

---

## 📌 Interview Talking Points — `theme.json` & Global Styles

**Q1. What problem do they solve?**
They centralize **design tokens + editor config** into one file. WP then manages CSS vars, cascade, and output — keeping themes consistent, performant, and user-customizable without hacks.

**Q2. Settings vs Styles?**

* **Settings** = tokens + capabilities (palettes, fonts, widths, spacing, on/off toggles).
* **Styles** = actual CSS rules, global or per block, built on those tokens.

**Q3. How do presets become CSS/UI?**
Declared in `settings` → WP generates **CSS vars** (`--wp--preset--*`) + **utility classes** (`.has-*`). Editor shows pickers wired to them, ensuring controlled choices.

**Q4. How do user changes interact with defaults?**
Cascade: **Core < Theme < Child Theme < User**. User edits in Global Styles (DB) override everything above, so designs stay editable without touching theme files.

**Q5. Style Variations vs Child Themes?**

* **Style Variations** = extra `/styles/*.json` “skins” (light/dark/brand). User choice saved as customization → updates won’t auto-apply.
* **Child Themes** = override *anything* (PHP, templates, styles). Variations = quick brand looks; child themes = deeper changes.

**Q6. What is the Style Engine?**
Core system that compiles `theme.json` + user settings → optimized CSS (editor + frontend). Guarantees consistent output. Public helpers (`wp_style_engine_get_styles()`) expose it.

**Q7. How to keep authors on-brand yet flexible?**

* Provide **strong presets** (palette, fonts, spacing, shadows).
* **Disable** off-brand controls (`color.custom: false`, locked units).
* **Scope settings per block** (e.g., lock headings).
* Offer **style variations** for approved brand looks.

👉 In short: **`theme.json` + Global Styles = design system + editor guardrails, compiled by the Style Engine into consistent CSS.**