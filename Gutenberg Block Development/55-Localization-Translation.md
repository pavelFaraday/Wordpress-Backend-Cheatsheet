# Internationalization & Localization of Gutenberg Blocks

## Why i18n matters (in Gutenberg too)

- **Internationalization (i18n)**: preparing your plugin/block so it _can_ be translated.
- **Localization (l10n)**: providing the actual translations for a locale (e.g., `ka_GE`, `de_DE`).
- In Gutenberg, strings can live in **three places**:

  1. **PHP** (server & render callbacks),
  2. **JavaScript** (editor UI, controls, notices),
  3. **`block.json`** (title, description, keywords, etc.).

Each layer has its own loading rules and file formats; getting all three right is the key.

---

## Core concepts (crucial details)

### 1) Text domain, headers, and paths

- **Text Domain**: unique slug for your plugin/block (e.g., `pixels-land`).
- **Headers (plugin main file)**:

  ```php
  /**
   * Plugin Name: Pixels Land Blocks
   * Text Domain: pixels-land
   * Domain Path: /languages
   */
  ```

- **Directory layout**:

  ```
  pixels-land/
    languages/
      pixels-land.pot
      pixels-land-ka_GE.po
      pixels-land-ka_GE.mo
      pixels-land-ka_GE-<hash>.json   // JS translations
    build/                            // compiled block JS/CSS
    src/                              // source JS
    block.json                        // metadata (translatable)
    pixels-land.php                    // main plugin file
  ```

### 2) PHP translations (MO/PO)

- Use core PHP functions: `__()`, `_e()`, `_x()`, `_n()`, `_nx()`, and escaped variants: `esc_html__()`, `esc_attr__()`.
- **Load PHP translations**:

  ```php
  add_action( 'init', function () {
      load_plugin_textdomain(
          'pixels-land',
          false,
          dirname( plugin_basename( __FILE__ ) ) . '/languages'
      );
  } );
  ```

### 3) JavaScript translations (JSON)

- Use `@wordpress/i18n` functions in JS: `__`, `_x`, `_n`, `_nx`, `sprintf`.

  ```js
  import { __, _x, _n, sprintf } from "@wordpress/i18n";

  const title = __("Image grid", "pixels-land");
  const msg = sprintf(
    _n("%d cube selected", "%d cubes selected", count, "pixels-land"),
    count
  );
  ```

- **Load JS translations** for a script handle:

  ```php
  // After wp_register_script(...) for your editor script:
  wp_set_script_translations(
      'pixels-land-editor', // your registered handle
      'pixels-land',
      plugin_dir_path( __FILE__ ) . 'languages'
  );
  ```

  WordPress will look for JSON files for the current locale and attach translations to that handle.

### 4) `block.json` is translatable

- Put `"textdomain": "pixels-land"` in `block.json`.
- Fields like `title`, `description`, `keywords` are extracted into the POT and localized.

  ```json
  {
    "apiVersion": 3,
    "name": "pixels-land/grid",
    "title": "Pixels Grid",
    "description": "Sell pixel cubes on a grid.",
    "category": "widgets",
    "textdomain": "pixels-land",
    "editorScript": "file:./build/index.js",
    "style": "file:./build/style-index.css",
    "viewScript": "file:./build/view.js",
    "keywords": ["pixels", "grid", "ads"]
  }
  ```

### 5) Extraction & build pipeline

- Use **WP-CLI i18n** tools (recommended) to generate POT and JSON:

  - `wp i18n make-pot` scans **PHP, JS, and block.json**.
  - `wp i18n make-json` converts PO → JSON for JavaScript locales.

- Optionally, the **Babel plugin** `@wordpress/babel-plugin-makepot` can auto-extract strings from JS during builds, but most teams standardize on WP-CLI for consistent PHP/JS/JSON extraction.

### 6) Translator comments & context

- Always add translator hints for placeholders, units, gender, etc.:

  ```php
  /* translators: %s is advertiser's brand name. */
  $msg = sprintf( __( 'Reserved by %s', 'pixels-land' ), $brand );
  ```

- Use `_x()` / `_nx()` to disambiguate homonyms via context.

### 7) Plurals and numbers

- Use `_n()` (PHP) / `_n()` (JS) for plural forms; don’t manual if/else the string.
- Format numbers and dates via i18n helpers:

  - PHP: `number_format_i18n()`, `wp_date()` (not deprecated like `date_i18n()`).
  - JS: `@wordpress/date`’s `dateI18n()` when rendering local dates in UI.

### 8) Performance & loading behavior

- `wp_set_script_translations()` loads only the **JSON** for that **script handle** and **current locale**—efficient and scoped.
- PHP translations (`.mo`) are loaded once per request via `load_plugin_textdomain()`.

---

# End-to-end example (minimal, realistic)

### PHP (plugin bootstrap)

```php
<?php
/**
 * Plugin Name: Pixels Land Blocks
 * Text Domain: pixels-land
 * Domain Path: /languages
 */

if ( ! defined( 'ABSPATH' ) ) exit;

add_action( 'init', function () {
    // 1) Load PHP translations (MO)
    load_plugin_textdomain( 'pixels-land', false, dirname( plugin_basename( __FILE__ ) ) . '/languages' );

    // 2) Register editor script (built asset)
    $asset = include plugin_dir_path( __FILE__ ) . 'build/index.asset.php';
    wp_register_script(
        'pixels-land-editor',
        plugins_url( 'build/index.js', __FILE__ ),
        $asset['dependencies'],
        $asset['version'],
        true
    );

    // 3) Attach JS translations (JSON)
    wp_set_script_translations(
        'pixels-land-editor',
        'pixels-land',
        plugin_dir_path( __FILE__ ) . 'languages'
    );

    // 4) Register block from metadata (block.json has "textdomain")
    register_block_type( __DIR__ . '/.' ); // or __DIR__ if block.json is at root
} );
```

### JS (editor UI)

```js
import { __, _n, sprintf } from "@wordpress/i18n";

export const PanelTitle = () =>
  __("Pixels Land – Grid Settings", "pixels-land");

export const SelectedCount = (count) =>
  sprintf(
    _n("%d cube selected", "%d cubes selected", count, "pixels-land"),
    count
  );
```

### `block.json`

```json
{
  "apiVersion": 3,
  "name": "pixels-land/grid",
  "title": "Pixels Grid",
  "description": "Sell pixel cubes on a grid.",
  "keywords": ["pixels", "grid", "ads"],
  "category": "widgets",
  "textdomain": "pixels-land",
  "editorScript": "pixels-land-editor",
  "style": "file:./build/style-index.css"
}
```

---

# Generating translation files (commands & workflow)

> **Pre-reqs**: WP-CLI installed in your dev environment.

1. **Create/refresh POT** (scan PHP, JS, and block.json):

```bash
wp i18n make-pot ./ ./languages/pixels-land.pot \
  --slug=pixels-land \
  --domain=pixels-land \
  --include=./
```

2. **Create/edit PO files** per locale with Poedit/Loco Translate/GlotPress:

- Example output: `languages/pixels-land-ka_GE.po`

3. **Compile MO files** (Poedit usually auto-compiles):

- `languages/pixels-land-ka_GE.mo`

4. **Generate JSON for JS** from your PO:

```bash
wp i18n make-json ./languages --no-purge
# Scans PO files and creates .json alongside them for JS
```

You’ll see files like:

```
languages/
  pixels-land-ka_GE.po
  pixels-land-ka_GE.mo
  pixels-land-ka_GE-<md5>.json
```

5. **Verify loading**

- Switch **Settings → General → Site Language** to your target locale.
- Open the block editor and confirm strings are translated.
- If PHP strings work but JS don’t, double-check:

  - `wp_set_script_translations()` handle matches your **registered handle**.
  - JSON files are in the `languages/` directory you pointed to.
  - Text domain in JS calls matches exactly (`'pixels-land'`).

---

# Practical use cases (Gutenberg-specific)

- **Inspector Controls / Toolbar labels**: `__( 'Reserve', 'pixels-land' )`
- **Notices & errors** shown in editor: `_n()` for plural counts, `sprintf` for dynamic bits.
- **Block transforms palette**: translatable labels to keep UX consistent across locales.
- **`block.json` metadata**: localized **title**, **description**, **keywords** for discoverability.
- **Server-rendered (dynamic) blocks**: translate on PHP side (render callback).
- **REST errors**: return translated messages if they are end-user facing (or return codes and translate on client, depending on API design).

---

# Gotchas & best practices

- **One text domain everywhere** (PHP, JS, block.json). Avoid typos/casing differences.
- **Never concatenate** translatable strings—use placeholders and `sprintf()`.
- **Always include translator comments** when placeholders or context might be unclear.
- **Don’t localize data** (e.g., usernames, URLs); localize only human-readable labels/messages.
- **Use plural APIs**; don’t hand-roll plural logic.
- **Escape after translation**: `esc_html__()` / `esc_attr__()` to avoid double-handling.
- **Keep source stable**: changing source strings invalidates existing translations.

---

# Quick Cheat-Sheet

**PHP**

```php
__( 'Text', 'pixels-land' );
_e( 'Echo', 'pixels-land' );
_x( 'Post', 'noun', 'pixels-land' );
_n( '%s cube', '%s cubes', $count, 'pixels-land' );
esc_html__( 'Safe HTML', 'pixels-land' );
```

**JS**

```js
import { __, _x, _n, _nx, sprintf } from "@wordpress/i18n";
__("Text", "pixels-land");
_x("Post", "noun", "pixels-land");
sprintf(_n("%d cube", "%d cubes", count, "pixels-land"), count);
```

**Loaders**

```php
load_plugin_textdomain( 'pixels-land', false, dirname( plugin_basename( __FILE__ ) ) . '/languages' );
wp_set_script_translations( 'pixels-land-editor', 'pixels-land', plugin_dir_path( __FILE__ ) . 'languages' );
```

**Commands**

```bash
wp i18n make-pot ./ ./languages/pixels-land.pot --slug=pixels-land --domain=pixels-land
wp i18n make-json ./languages --no-purge
```

**block.json**

```json
{ "textdomain": "pixels-land", "title": "Pixels Grid", "description": "..." }
```

---

# Interview-Style Talking Points (with strong model answers)

### 1) _What’s the difference between i18n and l10n in WordPress, especially for Gutenberg?_

**Answer:**
_i18n_ is preparing my plugin/block so every user-visible string is translatable across **PHP**, **JS**, and **block.json**. _l10n_ is providing the actual translations—`.po/.mo` for PHP and `.json` for JS. In Gutenberg, I load PHP translations via `load_plugin_textdomain()` and JS translations per-script via `wp_set_script_translations()`. I keep a single **text domain** across all layers to ensure a consistent lookup.

### 2) _How do you translate strings in block JavaScript and load those translations?_

**Answer:**
I use `@wordpress/i18n` functions like `__`, `_n`, `_x`, with the correct **text domain**. I register my editor script to get a handle (e.g., `pixels-land-editor`) and then call:

```php
wp_set_script_translations( 'pixels-land-editor', 'pixels-land', plugin_dir_path( __FILE__ ) . 'languages' );
```

This makes WordPress load the locale **JSON** file for that handle. I generate those JSON files from my PO files via `wp i18n make-json`.

### 3) _How do you handle translations in `block.json`?_

**Answer:**
I set `"textdomain": "pixels-land"` in `block.json`. Fields like `title`, `description`, and `keywords` are extracted into the POT when I run `wp i18n make-pot`. Translators provide PO files; WordPress will use them so the block is listed and searchable with localized metadata.

### 4) _Explain your end-to-end translation build pipeline._

**Answer:**

1. Write strings with `__`/`_n`/`_x` in PHP & JS; add translator comments where needed.
2. `wp i18n make-pot` to produce `languages/pixels-land.pot` scanning PHP, JS, and `block.json`.
3. Translators produce `*.po`, which compile to `*.mo` (PHP) and I run `wp i18n make-json` to create `*.json` for JS.
4. In code, I call `load_plugin_textdomain()` (PHP) and `wp_set_script_translations()` (JS).
5. I test by switching the site language and verifying UI messages, editor labels, and notices.

### 5) _When would you use `_n()` vs concatenation for plurals?_

**Answer:**
Always `_n()` (or `_nx()` for context). Plural rules differ by language; `_n()` lets WordPress pick the correct form for the locale. Concatenation breaks translation and pluralization.

### 6) _How do you ensure translators have enough context?_

**Answer:**
I add **translator comments** immediately above the string:

```php
/* translators: %s is an advertiser's brand name. */
sprintf( __( 'Reserved by %s', 'pixels-land' ), $brand );
```

In JS I do the same in comments near the call. If a word has multiple meanings, I use `_x()` to provide a context string.

### 7) _What’s the difference between `wp_localize_script` and `wp_set_script_translations`?_

**Answer:**
`wp_localize_script()` passes arbitrary PHP data (e.g., settings) into JS as a global. It’s **not** for translations. `wp_set_script_translations()` is specifically for **loading translation JSON** for a given script handle and text domain.

### 8) _Common pitfalls you watch out for?_

**Answer:**

- Mismatched **text domains** between PHP/JS.
- Forgetting `wp_set_script_translations()` for the editor handle.
- Missing or stale JSON files because `make-json` wasn’t run after updating PO.
- Concatenated strings and missing translator comments.
- Not escaping translated output appropriately (`esc_html__`, `esc_attr__`).

### 9) _How do you test and debug missing translations?_

**Answer:**
I switch **Site Language** in Settings and verify all UI text. For JS issues, I confirm the handle passed to `wp_set_script_translations()` matches the registered one, and that the JSON file exists under `languages/`. I also check that the exact source string (including punctuation) exists in the PO.

### 10) _Any Gutenberg-specific best practices?_

**Answer:**

- Localize **InspectorControl** labels, toolbar items, block placeholders, and notices.
- Keep `block.json` **title/description/keywords** translatable via the `"textdomain"` field.
- Keep all dynamic UI messages in JS translatable; don’t bake English into logic.
