# What are Block Patterns?

**Block Patterns** are _pre-built groups of blocks_ you can insert in one click (e.g., hero + CTA, pricing tables, testimonials). They drop **actual blocks** into the canvas that the user can freely edit—unlike synced patterns (formerly “Reusable blocks”), which are **linked instances** and update everywhere when edited. Patterns are discoverable from the **Patterns** tab in the inserter and the Pattern Explorer. ([WordPress Developer Resources][1])

**Key distinctions**

- **Patterns** – one-off starting layouts; once inserted they’re just normal blocks.
- **Synced patterns** – global, linked instances (edit in one place → updates everywhere).
- **Templates / Template Parts** – site structure for specific contexts; not user-insertable snippets.

---

# Why use Patterns?

- **Speed & consistency** for content creators.
- **Design governance** without locking content.
- **Contextual suggestions** (e.g., show certain patterns when inserting a `core/cover` block via `blockTypes`).
- **Targeting** by post type or template type to reduce noise in the inserter. (See the `postTypes` and `templateTypes` args below.) ([WordPress Developer Resources][2])

---

# Registering custom patterns (3 proven ways)

## 1) PHP: `register_block_pattern()` (plugin or theme)

Register patterns on the **`init`** hook. The `content` is **block HTML** (the same markup you get when copying blocks). You can also specify **categories**, **keywords**, **viewportWidth**, **blockTypes** (contextual display), **postTypes**, **templateTypes**, and even load **from a file** via `filePath` (WP 6.5+). ([WordPress Developer Resources][3])

```php
<?php
/**
 * Plugin Name: My Patterns
 */

add_action( 'init', function () {
    if ( ! function_exists( 'register_block_pattern' ) ) {
        return;
    }

    // 1) Optional: a custom category (shows only if at least one pattern uses it)
    register_block_pattern_category(
        'myplugin',
        [ 'label' => __( 'My Plugin Patterns', 'my-textdomain' ) ]
    );

    // 2) The pattern itself
    register_block_pattern(
        'myplugin/hero-cta',
        [
            'title'         => __( 'Hero with headline & CTA', 'my-textdomain' ),
            'description'   => __( 'Full-width hero with heading, lead and two buttons.', 'my-textdomain' ),
            'categories'    => [ 'myplugin', 'header', 'call-to-action' ],
            'keywords'      => [ 'hero', 'cover', 'cta' ],
            'viewportWidth' => 1200,
            'blockTypes'    => [ 'core/cover' ], // show in Cover’s contextual patterns
            'postTypes'     => [ 'page' ],
            'inserter'      => true,
            'content'       => <<<'HTML'
<!-- wp:cover {"overlayColor":"black","dimRatio":40,"minHeight":60,"minHeightUnit":"vh"} -->
<div class="wp-block-cover" style="min-height:60vh">
    <span aria-hidden="true" class="wp-block-cover__background has-black-background-color has-background-dim-40 has-background-dim"></span>
    <div class="wp-block-cover__inner-container">
        <!-- wp:heading {"textAlign":"center","level":1} -->
        <h1 class="has-text-align-center">Build faster with Patterns</h1>
        <!-- /wp:heading -->

        <!-- wp:paragraph {"align":"center","fontSize":"large"} -->
        <p class="has-text-align-center has-large-font-size">Drop-in hero section ready to customize.</p>
        <!-- /wp:paragraph -->

        <!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
        <div class="wp-block-buttons">
            <!-- wp:button -->
            <div class="wp-block-button"><a class="wp-block-button__link wp-element-button">Get started</a></div>
            <!-- /wp:button -->

            <!-- wp:button {"className":"is-style-outline"} -->
            <div class="wp-block-button is-style-outline"><a class="wp-block-button__link wp-element-button">Learn more</a></div>
            <!-- /wp:button -->
        </div>
        <!-- /wp:buttons -->
    </div>
</div>
<!-- /wp:cover -->
HTML
        ],
    );
} );
```

- Function reference: **`register_block_pattern()`** and **`register_block_pattern_category()`**. ([WordPress Developer Resources][3])
- Accepted arguments (including `blockTypes`, `postTypes`, `templateTypes`, `filePath` and `inserter`): see **`WP_Block_Patterns_Registry::register()`**. ([WordPress Developer Resources][2])

> Tip (WP 6.5+): If storing markup in a separate file, pass `'filePath' => __DIR__ . '/patterns/hero-cta.html'` instead of `'content'`. ([WordPress Developer Resources][2])

---

## 2) Block theme / classic theme: the `/patterns` folder (auto-registration)

Place files into your theme’s **`/patterns`** directory. Each file must start with a **header comment** describing the pattern (Title, Slug, Categories, etc.). WordPress auto-registers them. PHP is allowed in these files for dynamic values (e.g., asset URLs). ([WordPress Developer Resources][4])

**`/wp-content/themes/mytheme/patterns/hero-cta.php`**

```php
<?php
/**
 * Title: Hero with headline & CTA
 * Slug: mytheme/hero-cta
 * Description: Full-width hero with heading, lead and two buttons.
 * Categories: header, call-to-action
 * Keywords: hero, cover, cta
 * Block Types: core/cover
 * Viewport Width: 1200
 * Inserter: true
 */
?>
<!-- wp:cover {"overlayColor":"black","dimRatio":40} -->
<div class="wp-block-cover">
    <span aria-hidden="true" class="wp-block-cover__background has-black-background-color has-background-dim-40 has-background-dim"></span>
    <div class="wp-block-cover__inner-container">
        <!-- wp:heading {"textAlign":"center"} --><h2 class="has-text-align-center"><?php echo esc_html__( 'Your headline', 'my-textdomain' ); ?></h2><!-- /wp:heading -->
        <!-- wp:paragraph {"align":"center"} --><p class="has-text-align-center"><?php echo esc_html__( 'Support copy goes here.', 'my-textdomain' ); ?></p><!-- /wp:paragraph -->
    </div>
</div>
<!-- /wp:cover -->
```

Docs: **Registering Patterns** & **Using PHP in Patterns**. ([WordPress Developer Resources][5])

---

## 3) Insert a pattern programmatically (JS)

You can fetch registered patterns in the editor and insert their **content** as blocks:

```js
// In an editor script (enqueue for post editor)
const { select, dispatch } = wp.data;
const patterns = select("core").getBlockPatterns(); // array of patterns
const p = patterns.find((x) => x.name === "myplugin/hero-cta");
if (p) {
  dispatch("core/block-editor").insertBlocks(wp.blocks.parse(p.content));
}
```

Selectors `getBlockPatterns()` / `getBlockPatternCategories()` live in the **core data store**. Great for custom inserter UIs or to auto-seed layouts. ([WordPress Developer Resources][6])

> If you don’t want a pattern in the UI but still want to insert it via JS, set `'inserter' => false` in PHP. ([WordPress Developer Resources][2])

---

# Practical use cases (patterns that ship well)

- **Hero / CTA** sections (with/without media).
- **Feature grids** (2–6 columns with icons).
- **Pricing tables**.
- **Post headers** (cover image + meta + title).
- **Newsletter / lead capture** sections.

(Use `blockTypes` to surface contextual patterns—for example, show a “Cover: Post Header” pattern when the user inserts a `core/cover` block.) ([WordPress Developer Resources][2])

---

# Managing categories & visibility

- **Categories** help group patterns in the explorer. Register with `register_block_pattern_category()` and reference that slug in your pattern’s `categories` array. Categories only appear if at least one pattern uses them. ([WordPress Developer Resources][7])
- **Limit where patterns appear** using:

  - `postTypes` → show only in e.g. `page`.
  - `templateTypes` → show when editing specific template kinds.
  - `blockTypes` → show as contextual pattern suggestions for specific blocks. ([WordPress Developer Resources][2])

---

# Disabling unwanted patterns (housekeeping)

- **Remove bundled core patterns** in your theme:

  ```php
  add_action( 'after_setup_theme', function() {
      remove_theme_support( 'core-block-patterns' );
  } );
  ```

- **Disable remote directory patterns** (from wordpress.org/patterns):

  ```php
  add_filter( 'should_load_remote_block_patterns', '__return_false' );
  ```

Docs & guidance here. ([WordPress Developer Resources][8], [WPExplorer][9])

---

# Common pitfalls & tips

- **Register on `init`**. Registering elsewhere (e.g., `current_screen`) can fail. ([GitHub][10])
- **Content must be a string** (or provide `filePath` in 6.5+). If you pass an array/object, you’ll get “Pattern content must be a string.” ([WordPress Developer Resources][2])
- **Not showing up?** Check: category is spelled correctly, `inserter` isn’t `false`, you’re editing an allowed `postTypes`, and (for contextual surfacing) the `blockTypes` you chose supports contextual patterns. ([WordPress Developer Resources][2])
- **Translations**: localize `title`, `description`, and any literal text in content (you can build the `content` string with `sprintf()` + `__()` if needed).

---

# Quick cheat-sheet

**Core ideas**

- Patterns = pre-built block layouts; inserted as normal, editable blocks. ([WordPress Developer Resources][1])
- Great for speed, consistency, and contextual suggestions.

**Register (plugin/theme, PHP)**

```php
add_action('init', function () {
  register_block_pattern_category( 'mycat', [ 'label' => __( 'My Cat', 'td' ) ] );

  register_block_pattern( 'ns/slug', [
    'title'         => __( 'Pattern title', 'td' ),
    'description'   => __( 'Short help text', 'td' ),
    'categories'    => [ 'mycat' ],
    'keywords'      => [ 'cta', 'hero' ],
    'viewportWidth' => 1200,
    'blockTypes'    => [ 'core/cover' ],
    'postTypes'     => [ 'page' ],
    'templateTypes' => [ 'page', 'single' ],
    'inserter'      => true,
    'content'       => '<!-- wp:paragraph --><p>…</p><!-- /wp:paragraph -->',
    // or 'filePath'   => __DIR__ . '/patterns/hero-cta.html', // WP 6.5+
  ] );
});
```

Refs. ([WordPress Developer Resources][3])

**Theme `/patterns` file header**

```php
/**
 * Title: My Feature Grid
 * Slug: mytheme/feature-grid
 * Categories: mycat, text, columns
 * Keywords: features, grid
 * Block Types: core/columns
 * Viewport Width: 1200
 * Inserter: true
 * Description: Three-column feature grid with icons.
 */
```

Refs. ([WordPress Developer Resources][4])

**Programmatic insert (JS)**

```js
const { select, dispatch } = wp.data;
const pattern = select("core")
  .getBlockPatterns()
  .find((p) => p.name === "ns/slug");
if (pattern) {
  dispatch("core/block-editor").insertBlocks(wp.blocks.parse(pattern.content));
}
```

Ref. ([WordPress Developer Resources][6])

**Disable clutter**

```php
remove_theme_support( 'core-block-patterns' );
add_filter( 'should_load_remote_block_patterns', '__return_false' );
```

Refs. ([WordPress Developer Resources][8], [WPExplorer][9])

---

[1]: https://developer.wordpress.org/block-editor/reference-guides/block-api/block-patterns/?utm_source=chatgpt.com "Patterns – Block Editor Handbook | Developer.WordPress.org"
[2]: https://developer.wordpress.org/reference/classes/wp_block_patterns_registry/register/ "WP_Block_Patterns_Registry::register() – Method | Developer.WordPress.org"
[3]: https://developer.wordpress.org/reference/functions/register_block_pattern/?utm_source=chatgpt.com "register_block_pattern() – Function | Developer.WordPress.org"
[4]: https://developer.wordpress.org/themes/features/block-patterns/?utm_source=chatgpt.com "Block Patterns (Archived) – Theme Handbook"
[5]: https://developer.wordpress.org/themes/patterns/registering-patterns/?utm_source=chatgpt.com "Registering Patterns – Theme Handbook"
[6]: https://developer.wordpress.org/block-editor/reference-guides/data/data-core/?utm_source=chatgpt.com "WordPress Core Data – Block Editor Handbook"
[7]: https://developer.wordpress.org/reference/functions/register_block_pattern_category/?utm_source=chatgpt.com "register_block_pattern_category() – Function"
[8]: https://developer.wordpress.org/block-editor/how-to-guides/themes/theme-support/?utm_source=chatgpt.com "Theme Support – Block Editor Handbook"
[9]: https://www.wpexplorer.com/how-to-disable-wordpress-gutenberg-block-patterns/?utm_source=chatgpt.com "How to Disable WordPress Gutenberg Block Patterns"
[10]: https://github.com/WordPress/gutenberg/issues/40736?utm_source=chatgpt.com "Cannot register_block_pattern using current_screen hook. ..."
