# PHP Filters

“Gutenberg gives me **server-side hooks** to control *what blocks are available*, *how they register*, and *how they render*.

* I curate blocks with **`allowed_block_types_all`**.
* I change block config during registration with **`block_type_metadata`**, **`block_type_metadata_settings`**, or **`register_block_type_args`**.
* I control output with **`render_block`**, **`render_block_{name}`**, **`pre_render_block`**, and **`render_block_data`**.
  I prefer **theme.json/supports** for built-ins, and use hooks when I need logic (role, post type, template) or cross-block policies.”

---

# Hooks map in one glance

* **Curation (what can be inserted):**
  `allowed_block_types_all( $allowed, WP_Block_Editor_Context $ctx )`
* **Registration (change settings/supports/styles before editor sees them):**
  `block_type_metadata`, `block_type_metadata_settings`, `register_block_type_args`
* **Rendering (change or bypass HTML output):**
  `render_block`, `render_block_{name}`, `pre_render_block`, `render_block_data`
* **Inserter UX:**
  `block_categories_all` (add/reorder categories)

---

# High-value scenarios (with patterns you can recite)

### 1) Allow only specific blocks per post type / template / role

```php
add_filter('allowed_block_types_all', function ($allowed, $ctx) {
    if ( isset($ctx->post) && $ctx->post->post_type === 'product' ) {
        return [ 'core/heading', 'core/paragraph', 'core/image', 'woocommerce/handpicked-products' ];
    }
    return true; // elsewhere allow all
}, 10, 2);
```

**Why:** editorial control, brand consistency, simpler authoring.

---

### 2) Enforce supports globally (spacing, border, duotone) without editing each block

```php
add_filter('block_type_metadata_settings', function ($settings, $metadata) {
    $supports = $settings['supports'] ?? [];
    $supports['spacing']['margin']  = true;
    $supports['spacing']['padding'] = true;
    $supports['border']             = true;
    $settings['supports'] = $supports;
    return $settings;
}, 10, 2);
```

**Why:** standardize capabilities across core and custom blocks.

---

### 3) Per-block fine-tuning at the last mile

```php
add_filter('register_block_type_args', function ($args, $name) {
    if ($name === 'core/image') {
        $args['styles'][] = ['name' => 'rounded', 'label' => 'Rounded'];
        $args['supports']['align'] = ['wide', 'full'];
    }
    return $args;
}, 10, 2);
```

**Why:** adjust a single block without touching its source.

---

### 4) Modify output (light-touch) or suppress it conditionally

```php
// A) Light HTML tweak across all blocks
add_filter('render_block', function ($content, $block) {
    if ($block['blockName'] === 'core/paragraph') {
        return preg_replace('/^<p\b/', '<p class="is-highlighted"', $content, 1);
    }
    return $content;
}, 10, 2);

// B) Hide a specific block type in certain contexts
add_filter('pre_render_block', function ($pre, $parsed) {
    if ($parsed['blockName'] === 'core/quote' && is_search()) return '';
    return $pre;
}, 10, 2);
```

**Why:** quick compliance/brand tweaks; context-aware output.

---

# How to choose the right hook (say this out loud)

* **Need to limit what authors can insert?** `allowed_block_types_all`.
* **Want to change capabilities (supports) or styles before editor loads?**
  Prefer `block_type_metadata_settings` (bulk) or `register_block_type_args` (per-block).
* **Need to alter HTML output at runtime?** `render_block_{name}` (targeted) > `render_block` (global).
* **Need to skip default render based on conditions?** `pre_render_block`.

---

# Best practices (bullet points interviewers love)

* **Pick the earliest hook that fits.** Earlier = more consistent (registration hooks) vs later = more fragile (render surgery).
* **Don’t regex big HTML changes.** For real structure changes, prefer dynamic blocks (server `render_callback`) or client attributes + templates.
* **theme.json first.** Use hooks when you need logic (roles, post types, templates) or cross-block policies beyond theme.json’s static scope.
* **Keep `render_block` lean.** It runs for every block instance—use the specific variant when possible.
* **Guard by block name.** Never assume a hook only runs for your blocks.
* **Test in all editors.** Post Editor, Site Editor, Widgets—contexts differ.
* **Security:** treat block attrs as user input; escape on output when you inject attributes/HTML.

---

# Common pitfalls (and how you’d fix them)

* **“My JS filter doesn’t stick.”** A server hook is overriding it. Move the change to `block_type_metadata_settings` / `register_block_type_args`.
* **“Authors still see blocked blocks in Site Editor.”** You curated the Post Editor only; ensure you use `allowed_block_types_all` and respect `$ctx`.
* **“Class added at save not visible in editor.”** Mirror with a client filter (`editor.BlockListBlock`) or use a server `render_block` to guarantee runtime.

---

# Thematic ties you can mention

* **Capabilities & roles:** combine `allowed_block_types_all` with user roles for governance.
* **Design systems:** use registration hooks to enforce spacing/typography globally.
* **Performance:** prefer targeted render hooks; avoid heavy string manipulation.
* **Migrations:** these hooks let you adopt new core features (e.g., spacing) without forking blocks.

---

# Mini Q\&A (quick answers you can deliver)

**Q:** Difference between `block_type_metadata_settings` and `register_block_type_args`?
**A:** Both adjust registration, but `metadata_settings` works on the settings *derived from* `block.json` (great for bulk policies), while `register_block_type_args` is a final per-block override with the block’s name.

**Q:** When use `render_block` vs `render_block_core/xyz`?
**A:** Use the specific `render_block_{name}` to avoid global cost and reduce conditionals. Use global only for broad policies.

**Q:** Where do you prefer to enforce spacing/borders?
**A:** In `theme.json` if static; in `block_type_metadata_settings` if you need logic (e.g., enable for all except X).

**Q:** How to hide blocks for a single template?
**A:** `allowed_block_types_all`, check `$ctx->post`, and match `get_page_template_slug()`.

---

# 60-second whiteboard snippet (memorize this)

```php
// 1) Curate inserter
add_filter('allowed_block_types_all', function ($allowed, $ctx) {
    if ( isset($ctx->post) && $ctx->post->post_type === 'landing' ) {
        return ['core/heading','core/paragraph','core/buttons'];
    }
    return true;
}, 10, 2);

// 2) Enforce supports globally
add_filter('block_type_metadata_settings', function ($s, $m) {
    $s['supports']['spacing'] = ['margin' => true, 'padding' => true];
    $s['supports']['border']  = true;
    return $s;
}, 10, 2);

// 3) Targeted output tweak
add_filter('render_block_core/image', function ($html, $block) {
    return str_replace('<figure', '<figure class="has-shadow"', $html);
}, 10, 2);
```

---

# Quick cheat-sheet (printable)

* **Curation:** `allowed_block_types_all`
* **Registration:**

  * Early/raw: `block_type_metadata`
  * Derived/bulk: `block_type_metadata_settings`
  * Final/per-block: `register_block_type_args`
* **Rendering:**

  * Global: `render_block`
  * Targeted: `render_block_{name}`
  * Skip/replace: `pre_render_block`
  * Edit attrs pre-render: `render_block_data`
* **Inserter UX:** `block_categories_all`
* **Rule of thumb:** theme.json > registration hooks > rendering hooks.

---

Got it—you’re after the **server-side (PHP) hooks** that shape block behavior globally: things like `allowed_block_types_all`, `block_type_metadata`, `block_type_metadata_settings`, etc. Here’s a focused deep-dive with copy-paste examples and a quick cheat-sheet at the end.

---

## Why these hooks matter

These PHP hooks let you **curate the editor** and **reshape blocks** before they reach the editor or front end:

* **Curation**: allow/deny blocks per post type, template, user role, etc. (`allowed_block_types_all`). ([WordPress Developer Resources][1])
* **Registration-time surgery**: adjust block metadata/settings/supports for *any* block—core or custom—*without forking code* (`block_type_metadata`, `block_type_metadata_settings`, `register_block_type_args`). ([WordPress Developer Resources][2])
* **Render-time control**: modify output or even short-circuit rendering (`render_block`, dynamic `render_block_{name}`, `pre_render_block`, `render_block_data`). ([WordPress Developer Resources][3], [Make WordPress][4])
* **UX**: organize the inserter with custom categories (`block_categories_all`). ([WordPress Developer Resources][5])

---

# 1) Editor curation with `allowed_block_types_all`

**Goal:** Allow only a curated set of blocks for a given post type (and still keep all blocks elsewhere).

```php
add_filter( 'allowed_block_types_all', function( $allowed, $context ) {

    if ( isset( $context->post ) && $context->post->post_type === 'product' ) {
        // Whitelist for Woo products
        return array(
            'core/paragraph',
            'core/heading',
            'core/image',
            'core/list',
            'woocommerce/handpicked-products',
        );
    }

    // Default: allow everything
    return true;

}, 10, 2 );
```

* Returns `true` to allow all, `false` to block all, or an **array of block names** to whitelist.
* Works across editor contexts (post editor, site editor, widgets, etc.) via `WP_Block_Editor_Context`. ([WordPress Developer Resources][1])

> Note: older `allowed_block_types` is deprecated—use `_all`. ([WordPress Developer Resources][6])

---

# 2) Change metadata & supports during registration

## A) `block_type_metadata` — edit raw `block.json` before it’s processed

**Use when:** you want to override fields *as read from* `block.json` (e.g., `apiVersion`, `supports`).

```php
add_filter( 'block_type_metadata', function( $metadata ) {
    // Force apiVersion 2 for all blocks (example)
    $metadata['apiVersion'] = 2;
    return $metadata;
} );
```

This runs *very early*, before settings are derived. ([WordPress Developer Resources][7])

## B) `block_type_metadata_settings` — edit the **derived** settings

**Use when:** you need the fully built settings array that WP registers.

```php
add_filter( 'block_type_metadata_settings', function( $settings, $metadata ) {
    // Enable border support everywhere if missing
    $settings['supports']['border'] = true;

    // Example: add a style variation globally
    $settings['styles'][] = array( 'name' => 'polaroid', 'label' => 'Polaroid' );

    return $settings;
}, 10, 2 );
```

Great for bulk enabling supports or adding styles to many blocks. ([WordPress Developer Resources][8])

## C) `register_block_type_args` — final, low-level override per block

**Use when:** you want a **per-block** adjustment with access to the block’s name.

```php
add_filter( 'register_block_type_args', function( $args, $block_type ) {

    if ( $block_type === 'core/image' ) {
        // Add example style, tweak supports
        $args['styles'][] = array( 'name' => 'rounded', 'label' => 'Rounded' );
        $args['supports']['align'] = array( 'wide', 'full' );
    }

    return $args;

}, 10, 2 );
```

This is the most direct server hook and works for any server-registered block (core/custom). ([WordPress Developer Resources][9])

---

# 3) Tweak output at render time

You have three levels of control, from broad to surgical:

## A) `render_block` — filter content of **every** block

```php
add_filter( 'render_block', function( $content, $block, $instance ) {
    if ( $block['blockName'] === 'core/paragraph' ) {
        // Add a class to the first <p>
        $content = preg_replace( '/^<p\b/', '<p class="is-highlighted"', $content, 1 );
    }
    return $content;
}, 10, 3 );
```

Useful for light-touch transformations across many blocks. ([WordPress Developer Resources][3])

## B) `render_block_{name}` — target **one block type** only

```php
add_filter( 'render_block_core/quote', function( $content, $block ) {
    return '<div class="quote-wrap">'.$content.'</div>';
}, 10, 2 );
```

Introduced so you don’t need to `if` inside `render_block`. It only fires for the named block. ([Make WordPress][4])

## C) `pre_render_block` — **short-circuit** rendering

Return a string and WordPress will use it instead of the default render.

```php
add_filter( 'pre_render_block', function( $pre, $parsed_block ) {
    if ( $parsed_block['blockName'] === 'core/paragraph' && ! empty( $parsed_block['attrs']['hidden'] ) ) {
        return ''; // fully suppress output
    }
    return $pre; // null means "continue normal rendering"
}, 10, 2 );
```

Perfect when you need to suppress or replace output. ([WordPress Developer Resources][10])

## D) `render_block_data` — modify the parsed block **before** render

```php
add_filter( 'render_block_data', function( $parsed_block, $source_block ) {
    if ( $parsed_block['blockName'] === 'core/image' ) {
        $parsed_block['attrs']['className'] = trim(
            ($parsed_block['attrs']['className'] ?? '') . ' has-shadow'
        );
    }
    return $parsed_block;
}, 10, 2 );
```

This is great for altering attributes safely prior to HTML generation. ([WordPress Developer Resources][11])

---

# 4) Organize the inserter with `block_categories_all`

Add or reorder categories so your custom blocks are discoverable:

```php
add_filter( 'block_categories_all', function( $cats, $context ) {
    $custom = array(
        'slug'  => 'pixels-land',
        'title' => __( 'Pixels Land', 'your-textdomain' ),
        'icon'  => null,
    );
    // Insert after "media"
    $pos = array_search( 'media', array_column( $cats, 'slug' ), true );
    array_splice( $cats, $pos === false ? count( $cats ) : $pos + 1, 0, array( $custom ) );
    return $cats;
}, 10, 2 );
```

Works across contexts via `WP_Block_Editor_Context`. ([WordPress Developer Resources][5])

---

# Practical patterns you’ll actually use

### A) Lock down blocks per template or user role

```php
add_filter( 'allowed_block_types_all', function( $allowed, $context ) {
    $user = wp_get_current_user();

    // Restrict specific template
    if ( isset( $context->post ) && get_page_template_slug( $context->post ) === 'templates/landing.php' ) {
        return array( 'core/heading', 'core/paragraph', 'core/buttons' );
    }

    // Editors get everything, authors get curated set
    if ( in_array( 'author', (array) $user->roles, true ) ) {
        return array( 'core/paragraph', 'core/image', 'core/list' );
    }

    return true;
}, 10, 2 );
```

([WordPress Developer Resources][1])

### B) Enforce supports globally (e.g., borders/duotone/spacing)

```php
add_filter( 'block_type_metadata_settings', function( $settings, $metadata ) {
    $supports = $settings['supports'] ?? array();
    $supports['spacing']['margin'] = true;
    $supports['spacing']['padding'] = true;
    $supports['border'] = true;
    $settings['supports'] = $supports;
    return $settings;
}, 10, 2 );
```

([WordPress Developer Resources][8])

### C) Per-block override with `register_block_type_args`

```php
add_filter( 'register_block_type_args', function( $args, $block_type ) {
    if ( $block_type === 'core/media-text' ) {
        $args['supports']['__experimentalDuotone'] = true; // example
    }
    return $args;
}, 10, 2 );
```

([WordPress Developer Resources][9])

### D) Replace output for a block if a condition is met

```php
add_filter( 'pre_render_block', function( $pre, $parsed_block ) {
    if ( $parsed_block['blockName'] === 'core/quote' && is_search() ) {
        return ''; // hide quotes on search results
    }
    return $pre;
}, 10, 2 );
```

([WordPress Developer Resources][10])

---

# Caveats & tips

* **Order of operations:** `block_type_metadata` → derive settings → `block_type_metadata_settings` → register → `register_block_type_args`. Pick the hook that matches your intent and granularity. ([WordPress Developer Resources][7])
* **Client vs server precedence:** Server-side settings propagate to the client with **higher priority** than client-side tweaks. If a client filter “doesn’t stick,” check server filters. ([WordPress Developer Resources][7])
* **Performance:** `render_block` runs for *every* block instance. Keep callbacks lean; prefer targeted `render_block_{name}` where possible. ([Make WordPress][4])
* **Deprecations:** Use `allowed_block_types_all`, not `allowed_block_types`. ([WordPress Developer Resources][6])

---

# Quick cheat-sheet

**Curation**

* `allowed_block_types_all( $allowed, WP_Block_Editor_Context $ctx )` → return `true|false|string[]`. Scope by post type/template/role. ([WordPress Developer Resources][1])

**Registration**

* `block_type_metadata( array $metadata )` → edit raw `block.json` before processing. ([WordPress Developer Resources][2])
* `block_type_metadata_settings( array $settings, array $metadata )` → edit derived settings (supports, styles). ([WordPress Developer Resources][8])
* `register_block_type_args( array $args, string $block_type )` → final per-block override. ([WordPress Developer Resources][9])

**Rendering**

* `render_block( string $content, array $block, WP_Block $instance )` → filter HTML of any block. ([WordPress Developer Resources][3])
* `render_block_{name}( string $content, array $block )` → filter HTML of a **specific** block. ([Make WordPress][4])
* `pre_render_block( ?string $pre, array $parsed_block, WP_Block|null $parent )` → short-circuit rendering by returning a string. ([WordPress Developer Resources][10])
* `render_block_data( array $parsed_block, array $source_block, WP_Block|null $parent )` → adjust attributes before render. ([WordPress Developer Resources][11])

**Inserter UX**

* `block_categories_all( array $cats, WP_Block_Editor_Context $ctx )` → add/reorder categories. ([WordPress Developer Resources][5])

---

If you tell me the exact blocks and rules you want (e.g., *“on Products, allow only X/Y, and add border support to Images”*), I’ll stitch these into ready-to-drop plugin code that fits your Pixels Land ecosystem.

[1]: https://developer.wordpress.org/reference/hooks/allowed_block_types_all/?utm_source=chatgpt.com "allowed_block_types_all – Hook | Developer.WordPress.org"
[2]: https://developer.wordpress.org/reference/hooks/block_type_metadata/?utm_source=chatgpt.com "block_type_metadata – Hook | Developer.WordPress.org"
[3]: https://developer.wordpress.org/reference/hooks/render_block/?utm_source=chatgpt.com "render_block – Hook - WordPress Developer Resources"
[4]: https://make.wordpress.org/core/2021/02/18/wordpress-5-7-a-new-dynamic-hook-to-filter-the-content-of-a-single-block/?utm_source=chatgpt.com "A new dynamic hook to filter the content of a single block"
[5]: https://developer.wordpress.org/reference/hooks/block_categories_all/?utm_source=chatgpt.com "block_categories_all – Hook - WordPress Developer Resources"
[6]: https://developer.wordpress.org/reference/hooks/allowed_block_types/?utm_source=chatgpt.com "allowed_block_types – Hook - WordPress Developer Resources"
[7]: https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/?utm_source=chatgpt.com "Block Filters – Block Editor Handbook"
[8]: https://developer.wordpress.org/reference/hooks/block_type_metadata_settings/?utm_source=chatgpt.com "block_type_metadata_settings – Hook"
[9]: https://developer.wordpress.org/reference/hooks/register_block_type_args/?utm_source=chatgpt.com "register_block_type_args – Hook | Developer.WordPress.org"
[10]: https://developer.wordpress.org/reference/hooks/pre_render_block/?utm_source=chatgpt.com "pre_render_block – Hook - WordPress Developer Resources"
[11]: https://developer.wordpress.org/reference/hooks/render_block_data/?utm_source=chatgpt.com "render_block_data – Hook - WordPress Developer Resources"
