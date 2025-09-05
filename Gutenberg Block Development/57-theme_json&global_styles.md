# theme.json & Global Styles in Gutenberg — Deep Dive + Interview Prep

## What they are (in one line)

**theme.json** is your theme’s single source of truth for editor capabilities (what controls users see) and design tokens/styles (what the site looks like). **Global Styles** is the UI (in the Site Editor) that reads/writes those values and merges them with user choices. ([WordPress Developer Resources][1])

## Styles Flow

1️⃣ Block supports → Blocks declare what styling features they allow (color, typography, spacing, etc.)
2️⃣ theme.json (settings) → Theme enables/disables features & defines design tokens (palette, fonts, spacing scale)
3️⃣ CSS variables → WP generates --wp--preset--_ vars + utility classes (.has-_), based on theme.json
4️⃣ Editor UI → Blocks show palettes, font pickers, spacing controls (as allowed by block supports + theme.json)
5️⃣ User Global Styles → Site Editor overrides theme defaults; saved in DB (highest priority in cascade)
6️⃣ Final Styles → Style Engine outputs optimized CSS for editor + frontend (consistent & performant)

---

## Why they exist (purpose & benefits)

- **One place to configure the editor**: enable/disable features, define palettes, fonts, spacing scales, layout widths, etc. (instead of many `add_theme_support()` flags or scattered CSS). ([WordPress Developer Resources][1])
- **Managed CSS output**: WordPress generates CSS variables & classes from your presets and emits only what’s needed, reducing conflicts and specificity wars. ([WordPress Developer Resources][1])
- **User-first editing**: Site owners can adjust styles globally in the UI; their choices override theme defaults in a clear hierarchy. ([WordPress Developer Resources][2])

---

## The mental model (hierarchy & merge order)

Four “origins,” lowest to highest priority:

1. **Core defaults** → 2) **Theme `theme.json`** → 3) **Child theme `theme.json`** → 4) **User Global Styles** (stored in DB). Higher levels win. ([WordPress Developer Resources][2])

---

## The file & versions (what to ship)

- Lives at the **root of your theme** as `theme.json`.
- Use **schema v3** for WordPress **6.6+**; point `$schema` to the minimum WP you support (e.g. `https://schemas.wp.org/wp/6.6/theme.json`). v2 still works but new features land in v3. ([WordPress Developer Resources][3], [Make WordPress][4])

---

## Core concepts (settings vs styles)

### 1) `settings` = what the editor allows + your **design tokens**

- Toggle UI controls (e.g., `color.custom`, `typography.customFontSize`, `border.radius`, `spacing.units`).
- Define **presets** that become CSS variables & classes:

  - **Color** (`palette`, `gradients`, `duotone`)
  - **Typography** (`fontFamilies`, `fontSizes`, optional fluid)
  - **Spacing** (`spacingSizes` or auto `spacingScale`)
  - **Shadows**, **border radius sizes**, **aspect ratios**

- You can scope settings **per block** to fine-tune what each block exposes. ([WordPress Developer Resources][3])

**Preset outputs** (examples):

- Variables like `--wp--preset--color--brand` and utility classes like `.has-brand-color`. ([WordPress Developer Resources][1])

**Custom variables**: a top-level `settings.custom` object creates your own `--wp--custom--*` CSS vars (nested keys map to `--` segments). ([WordPress Developer Resources][3])

Popular settings to know cold:

- `layout.contentSize` / `layout.wideSize` (site widths, fluid typography max viewport)
- `typography.fluid` (global fluid type parameters)
- `spacing.blockGap`, `spacing.spacingScale` (global spacing)
- `appearanceTools` (one switch to expose a broad set of UI controls) ([WordPress Developer Resources][3])

### 2) `styles` = **the CSS** (root or per-block)

- Mirrors CSS concerns: `color`, `typography`, `spacing`, `border`, `shadow`, `background`, `filter/duotone`, etc.
- Top-level `styles` apply to the site (`body`), and `styles.blocks["core/paragraph"]` targets a block.
- You can include a `styles.css` string for edge cases not covered by keys. ([WordPress Developer Resources][3])

---

## The Style Engine (how CSS is produced)

WordPress’ **Style Engine** turns your theme.json and user choices into variables, classes, and compiled CSS—consistently for editor and frontend. Public helpers like `wp_style_engine_get_styles()` reflect how this compilation works; core uses them under the hood. ([WordPress Developer Resources][5])

---

## Global Styles UI (what users see)

In the Site Editor, users change typography, colors, spacing, etc. Those choices are saved as **user-level styles** that override your theme.json defaults according to the hierarchy above. ([WordPress Developer Resources][2])

---

## Style Variations (productizing “skins”)

Ship multiple **`/styles/*.json`** files—each is a full or partial theme.json “skin.” Users pick a variation; its data is stored as a user customization (so updates to the variation won’t apply if the user already saved changes). Great for light/dark or brand flavors. ([WordPress Developer Resources][6])

---

## Practical use cases (what teams actually do)

- **Design tokens**: lock down a palette, spacing scale, typography families/sizes (w/ fluid type), shadow presets—then allow limited customization. ([WordPress Developer Resources][3])
- **Editor hardening**: disable custom colors or units, restrict features per block (e.g., headings can’t change alignment). ([WordPress Developer Resources][1])
- **Layout rails**: define content/wide widths; use root-padding-aware alignments to keep full-width blocks inside safe padding. ([WordPress Developer Resources][3])
- **Consistent spacing**: opt into `blockGap` and set a site-wide rhythm; blocks that support layout translate it to margins/gaps. ([WordPress Developer Resources][1])
- **Brand packages**: ship style variations (corporate, campaign, seasonal). ([WordPress Developer Resources][6])

---

## Quick cheat-sheet (copyable, high-level)

**Top-level keys**

- `version` (use **3** for WP 6.6+)
- `settings` → editor capabilities + tokens (palettes, fonts, spacing, layout)
- `styles` → CSS for root & blocks
- `customTemplates`, `templateParts`, `patterns` (metadata) ([WordPress Developer Resources][3])

**Must-know settings**

- `layout.contentSize` / `layout.wideSize`
- `typography.fontFamilies`, `typography.fontSizes`, `typography.fluid`
- `color.palette`, `color.gradients`, `color.duotone`
- `spacing.spacingSizes` / `spacing.spacingScale`, `spacing.blockGap`
- `shadow.presets`, `border.radiusSizes`
- `appearanceTools: true` (one-switch to expose key controls) ([WordPress Developer Resources][3])

**Per-block control**

- `settings.blocks["core/heading"].typography.textAlign = false` (example idea)
- `styles.blocks["core/button"].color.background = "var(--wp--preset--color--brand)"` (use your tokens) ([WordPress Developer Resources][1])

**Variables & utilities**

- Preset var: `--wp--preset--color--{slug}`; class: `.has-{slug}-color`
- Custom var: `--wp--custom--{key}--{nested-key}` (from `settings.custom`) ([WordPress Developer Resources][1])

**Style variations**

- Put JSON files in `/styles/*.json`; users choose in Styles UI; acts like a “skin.” ([WordPress Developer Resources][6])

**Schema**

- Add `$schema` matching your **minimum WP version** to get editor hints & avoid using too-new keys. ([WordPress Developer Resources][3], [Make WordPress][4])

**Caching tip**

- theme.json is cached; during dev, set `WP_DEBUG` or `SCRIPT_DEBUG` true to see changes immediately. ([WordPress Developer Resources][1])

---

## Common pitfalls (and quick fixes)

- **Wrong namespace or version** → Use **v3** keys on WP 6.6+, and set `$schema` to the correct WP version. ([WordPress Developer Resources][3], [Make WordPress][4])
- **“My tokens don’t show in the editor”** → Presets must be under `settings` (not `styles`), and blocks must support the feature. ([WordPress Developer Resources][1])
- **Spacing confusion** → `spacing.blockGap` has both a setting (boolean/null to opt in/out of UI & output) and a value in `styles.spacing.blockGap`. Know both. ([WordPress Developer Resources][1])
- **Expecting variations to update live** → Once a user selects/edits a variation, it becomes user data; they won’t get your later variation updates automatically. ([WordPress Developer Resources][6])

---

## Interview-style talking points (with strong model answers)

**Q1. What problem do theme.json & Global Styles solve?**
**A.** They centralize editor configuration and design tokens into one declarative file. WordPress then **manages CSS** for you—creating variables/classes, handling cascade between core/theme/user, and emitting only necessary CSS—so themes stay consistent and user-customizable without specificity hacks. ([WordPress Developer Resources][1])

**Q2. Explain “settings vs styles.”**
**A.** **Settings** define capabilities and tokens (palettes, fonts, spacing units, layout widths) and can be scoped per block. **Styles** apply actual CSS at root or per-block using those tokens. Settings power the editor UI; styles shape the output. ([WordPress Developer Resources][3])

**Q3. How do presets become CSS and UI?**
**A.** When you declare palettes, gradients, font sizes, spacing scales in `settings`, WordPress generates **CSS variables** (`--wp--preset--…`) and **utility classes** (`.has-…`). The editor shows UI pickers wired to them, giving users controlled choices. ([WordPress Developer Resources][1])

**Q4. How do user changes interact with theme defaults?**
**A.** There’s a **hierarchy**: core < theme < child theme < user. Edits in the Global Styles UI are stored in the DB and **override** theme.json. That’s why designs remain editable without modifying theme files. ([WordPress Developer Resources][2])

**Q5. What’s Style Variations vs. Child Themes?**
**A.** Variations are alternate **theme.json files** in `/styles/*.json` that act like **skins**. When selected, they’re saved as user customizations (so updates don’t auto-apply if the user already saved). Child themes can override anything, not just styles. Use variations for quick brand looks; child themes for broader changes. ([WordPress Developer Resources][6])

**Q6. What changed with theme.json version 3?**
**A.** v3 (WP 6.6+) is the **current spec**; use the matching `$schema` for your minimum WP version. New features and keys land in v3 going forward; v2 remains supported but not extended. ([WordPress Developer Resources][3])

**Q7. How do you control spacing across a site?**
**A.** Use `spacing.spacingScale` or `spacing.spacingSizes` to define a rhythm; enable `spacing.blockGap` (setting) and set `styles.spacing.blockGap` for the value. Blocks that support layout translate it to consistent gaps/margins between blocks. ([WordPress Developer Resources][3])

**Q8. What is the Style Engine and why should I care?**
**A.** It’s the system core uses to compile theme.json + user settings into CSS for editor and frontend. It ensures consistent output and smaller CSS. Public helpers like `wp_style_engine_get_styles()` mirror this logic. ([WordPress Developer Resources][5])

**Q9. How do you keep authors “on-brand” while still flexible?**
**A.** Provide strong **presets** (palette, fontFamilies, spacingScale, shadows), **disable** off-brand options (`color.custom: false`, locked units), and scope **per-block settings** to limit controls where necessary. Then expose style variations for brand themes. ([WordPress Developer Resources][3])

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

# 3. 🚀 Why It’s Crucial for WordPress 6+ Era Development

- **Future-proofing**: theme.json is the **official standard**. Relying on CSS-only themes or `add_theme_support()` is legacy.
- **Global Styles system**: users can override theme.json values in the Site Editor → your theme must integrate cleanly.
- **Consistency**: By defining tokens (palette, type scale, spacing), every block stays on-brand, frontend = editor.
- **Performance**: WordPress only outputs the CSS you need (managed by the Style Engine), reducing bloat.
- **Customization**: Style Variations (different theme.json presets in `/styles/`) let you ship multiple “skins” without extra themes.

👉 In interviews, the key is to emphasize:

> “In the WP 6+ era, `theme.json` + block supports = the foundation of theme development. It centralizes design tokens, controls the editor UI, and ensures consistency between blocks, site editor, and frontend — while letting users customize through Global Styles.”

---

✅ Would you like me to create a **visual one-pager (diagram)** showing:
**block supports → theme.json settings → CSS variables → block UI → final styles**? That could be a powerful cheat-sheet for interviews.
