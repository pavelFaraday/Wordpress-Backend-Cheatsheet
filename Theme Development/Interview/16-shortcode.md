# 📌 WordPress Shortcodes: A Comprehensive Deep Dive

### What Are Shortcodes?

Shortcodes **allow you to add dynamic content or functionality to posts, pages, and widgets <u>without writing complex PHP code</u>. They act as macros that expand into more complex content when the page is rendered.** It is a small tag-like piece of text you put in post/page content and WordPress will replace that tag with dynamic content when the post is shown.

### Purpose of Shortcodes

- **❗️Code reusability**: One piece of code that can be used multiple times in many posts/pages.
- **❗️Simplify content creation**: Enable non-technical users to add complex features
- **❗️Consistency**: Maintain uniform implementation of features across a site
- **❗️Quick dynamic content:** show lists of posts, forms, buttons, widgets, widgets in content areas.
- **❗️Editor friendly**: content editors can add UI/functionality without PHP
- **❗️Separation of concerns**: Keep PHP logic separate from content, content remains clean.

### Fundamentals

1. There are 2 types of shortcodes: **Self-closing vs. Enclosing Shortcodes**:

   - Self-closing: `[gallery ids="1,2,3"]`
   - Enclosing: `[box]Content here[/box]`

2. **Attributes**: Key-value pairs that configure the shortcode behavior

   - Example: `[button color="blue" size="large"]Click me[/button]`

3. **Callback Signature:** `function callback_name($atts = [], $content = null, $tag = '') { ... }`

4. **Registration:**
   `add_shortcode( 'tag', 'callback' );`
   `remove_shortcode( 'tag' );`
   `shortcode_exists( 'tag' );`

5. **Return vs echo:** Always return the generated string❗️ Don’t echo — WP expects a return❗️
6. **❗️Register assets once; enqueue when shortcode runs❗️**
7. **Nesting / enclosed content:** If your shortcode encloses content, use `$content` and `do_shortcode( $content)` to process nested shortcodes.
8. **Where shortcodes are processed:** Shortcodes are expanded when `do_shortcode()` is called (common path: `the_content` filter). If you want them in custom HTML, call `do_shortcode()` manually.

### Common Use Cases in Practice

1. **Embedding Media**: `[video]`, `[audio]`, `[gallery]`
2. **Displaying Dynamic Content**: Recent posts, popular products, testimonials
3. **Creating Layout Elements**: Columns, tabs, accordions, buttons
4. **Forms and CTAs**: Contact forms, newsletter signups
5. **E-commerce Features**: Product displays, shopping carts
6. **Custom Functionality**: Maps, sliders, social media feeds

### Core Concepts

1. **Shortcode API**: WordPress's built-in system for registering and handling shortcodes
2. **Security:** Sanitize attributes (`sanitize_text_field`, `esc_url`, `intval`), escape output (`esc_html`, `esc_attr`, `wp_kses_post`) depending on type of output.
3. **Assets:** Register scripts/styles on init, enqueue them from inside the shortcode callback (so they load only when shortcode is used).
4. **Performance:** Avoid heavy queries inside shortcode on every page load

### Example:

**Register your shortcode**:
Usage: `[my_shortcode color="red" size="large"]This is my content[/my_shortcode]`

```php
function my_custom_shortcode($atts, $content = null) {
    // Extract attributes or set defaults
    $atts = shortcode_atts(
        array(
            'color' => 'blue',
            'size' => 'medium'
        ),
        $atts,
        'my_shortcode'
    );

    // Process the shortcode
    $output = '<div class="my-shortcode" style="color: ' . esc_attr($atts['color']) . '; font-size: ' . esc_attr($atts['size']) . ';">';
    $output .= do_shortcode($content); // Process nested shortcodes
    $output .= '</div>';

    return $output;
}
add_shortcode('my_shortcode', 'my_custom_shortcode');
```

 **Shortcode that outputs a button and enqueues a script only when used:**
   Usage: `[cta url="https://example.com" target="_blank"]Learn more[/cta]`

```php
// Register script/style on init
function ds_register_assets() {
    wp_register_script( 'ds-cta-js', plugin_dir_url(__FILE__) . 'assets/cta.js', array('jquery'), '1.0', true );
    wp_register_style( 'ds-cta-css', plugin_dir_url(__FILE__) . 'assets/cta.css', array(), '1.0' );
}
add_action( 'wp_enqueue_scripts', 'ds_register_assets' );

function ds_cta_shortcode( $atts, $content = null ) {
    $atts = shortcode_atts( array(
        'url'    => '#',
        'target' => '_self',
    ), $atts, 'cta' );

    // enqueue only when shortcode renders
    wp_enqueue_script( 'ds-cta-js' );
    wp_enqueue_style( 'ds-cta-css' );

    $url = esc_url( $atts['url'] );
    $target = esc_attr( $atts['target'] );

    return '<a class="ds-cta" href="' . $url . '" target="' . $target . '">' . esc_html( $content ? $content : 'Click' ) . '</a>';
}
add_shortcode( 'cta', 'ds_cta_shortcode' );
```

### Quick cheat-sheet (copyable)

- Register: `add_shortcode( 'tag', 'callback' )`;
- Callback signature: `function cb( $atts = [], $content = null, $tag = '' ) {}`
- Merge attrs: `$atts = shortcode_atts( $defaults, $atts, $tag )`;
- Nested: `$content = do_shortcode( $content )`;
- Return HTML, escape output: `esc_html()`, `esc_url()`, `wp_kses_post()`
- Enqueue assets from callback: `wp_enqueue_script()` / `wp_enqueue_style()`
- Cache heavy results: `set_transient()` / `get_transient()`
- Use `do_shortcode($content)` for nested shortcodes.

---
---
---

### Best practices

- Prefix shortcode names and functions (e.g., ds*, myplugin*) to avoid collisions.
- Always return; never echo.
- Sanitize attributes and escape output.
- Register assets once; enqueue when shortcode runs.
- Cache heavy output (transient) and invalidate when needed.


### Debugging & testing tips

- Enable `WP_DEBUG` and check `debug.log`.
- Use `shortcode_exists('tag')` to check registration.
- Test nested shortcodes in the editor with the Shortcode block or in a classic editor.
- If shortcode output is blank: make sure the function returns (not echoes), check for fatal errors, and check for name collisions.

### When NOT to use shortcodes

- For complex editor UIs and richer WYSIWYG editing, prefer Gutenberg blocks (blocks give better UX and visual editing).
- If the content is purely structural and belongs in a template, implement in theme templates instead.

---

## Step-by-Step Guide to Creating a Shortcode

### Method 1: Adding to functions.php

1. **Access your theme's functions.php file**
   Usage: `[hello name="Luka"]`

```php
// 1. Callback
function ds_hello_shortcode( $atts ) {
    $atts = shortcode_atts( array(
        'name' => 'friend',
    ), $atts, 'hello' );

    // sanitize and return — DO NOT echo
    return 'Hello, ' . esc_html( $atts['name'] ) . '!';
}
// 2. Register
add_shortcode( 'hello', 'ds_hello_shortcode' );
```

2. **Register your shortcode**:
   Usage: `[my_shortcode color="red" size="large"]This is my content[/my_shortcode]`

```php
function my_custom_shortcode($atts, $content = null) {
    // Extract attributes or set defaults
    $atts = shortcode_atts(
        array(
            'color' => 'blue',
            'size' => 'medium'
        ),
        $atts,
        'my_shortcode'
    );

    // Process the shortcode
    $output = '<div class="my-shortcode" style="color: ' . esc_attr($atts['color']) . '; font-size: ' . esc_attr($atts['size']) . ';">';
    $output .= do_shortcode($content); // Process nested shortcodes
    $output .= '</div>';

    return $output;
}
add_shortcode('my_shortcode', 'my_custom_shortcode');
```

3. **Shortcode that outputs a button and enqueues a script only when used:**
   Usage: `[cta url="https://example.com" target="_blank"]Learn more[/cta]`

```php
// Register script/style on init
function ds_register_assets() {
    wp_register_script( 'ds-cta-js', plugin_dir_url(__FILE__) . 'assets/cta.js', array('jquery'), '1.0', true );
    wp_register_style( 'ds-cta-css', plugin_dir_url(__FILE__) . 'assets/cta.css', array(), '1.0' );
}
add_action( 'wp_enqueue_scripts', 'ds_register_assets' );

function ds_cta_shortcode( $atts, $content = null ) {
    $atts = shortcode_atts( array(
        'url'    => '#',
        'target' => '_self',
    ), $atts, 'cta' );

    // enqueue only when shortcode renders
    wp_enqueue_script( 'ds-cta-js' );
    wp_enqueue_style( 'ds-cta-css' );

    $url = esc_url( $atts['url'] );
    $target = esc_attr( $atts['target'] );

    return '<a class="ds-cta" href="' . $url . '" target="' . $target . '">' . esc_html( $content ? $content : 'Click' ) . '</a>';
}
add_shortcode( 'cta', 'ds_cta_shortcode' );
```

4. **Shortcode that runs a WP_Query (with transient caching)**
   Usage: `[latest_posts count="3" category="news"]`

```php
function ds_latest_posts_shortcode( $atts ) {
    $atts = shortcode_atts( array(
        'count' => 3,
        'category' => '',
    ), $atts, 'latest_posts' );

    $count = intval( $atts['count'] );
    $cache_key = 'ds_latest_posts_' . md5( $count . '|' . $atts['category'] );
    if ( $html = get_transient( $cache_key ) ) {
        return $html;
    }

    $args = array(
        'posts_per_page' => $count,
        'post_status' => 'publish',
    );
    if ( ! empty( $atts['category'] ) ) {
        $args['category_name'] = sanitize_text_field( $atts['category'] );
    }

    $q = new WP_Query( $args );
    if ( ! $q->have_posts() ) {
        return '<p>No posts found.</p>';
    }

    ob_start();
    echo '<ul class="ds-latest-posts">';
    while ( $q->have_posts() ) {
        $q->the_post();
        printf(
            '<li><a href="%s">%s</a></li>',
            esc_url( get_permalink() ),
            esc_html( get_the_title() )
        );
    }
    echo '</ul>';
    wp_reset_postdata();

    $html = ob_get_clean();
    // cache for 10 minutes
    set_transient( $cache_key, $html, 10 * MINUTE_IN_SECONDS );

    return $html;
}
add_shortcode( 'latest_posts', 'ds_latest_posts_shortcode' );
```

---

### Method 2: Creating a Plugin for Your Shortcode

1. **Create a new PHP file** in `/wp-content/plugins/` (e.g., `my-shortcodes.php`)

2. **Add the plugin header**:

   ```php
   <?php
   /*
   Plugin Name: My Custom Shortcodes
   Description: Adds custom shortcodes to WordPress
   Version: 1.0
   Author: Your Name
   */
   ```

3. **Add your shortcode function** (same as above)

4. **Activate the plugin** through the WordPress admin panel

### Advanced Shortcode Techniques

1. **Nested Shortcodes**:

   ```php
   function parent_shortcode($atts, $content = null) {
       return '<div class="parent">' . do_shortcode($content) . '</div>';
   }
   add_shortcode('parent', 'parent_shortcode');

   function child_shortcode($atts) {
       return '<span class="child">Child content</span>';
   }
   add_shortcode('child', 'child_shortcode');
   ```

   Usage: `[parent][child][/parent]`

2. **Shortcodes with AJAX**:

   ```php
   function ajax_shortcode() {
       wp_enqueue_script('my-ajax-script');
       return '<div class="ajax-content" data-action="my_ajax_action">Loading...</div>';
   }
   add_shortcode('ajax_content', 'ajax_shortcode');
   ```

3. **Shortcode UI for Users**:
   - Use plugins like Shortcake (Shortcode UI) to create user-friendly interfaces
   - Or build custom meta boxes with TinyMCE integration

## Best Practices

1. **Security Considerations**:

   - Always sanitize and escape output (`esc_attr()`, `esc_html()`)
   - Validate and sanitize input data
   - Use nonces for shortcodes that perform actions

2. **Performance Optimization**:

   - Cache shortcode output when possible
   - Minimize database queries within shortcodes
   - Consider using transients for expensive operations

3. **Naming Conventions**:

   - Use unique prefixes to avoid conflicts
   - Be descriptive but concise
   - Follow WordPress coding standards

4. **Documentation**:
   - Include clear usage examples in code comments
   - Create documentation for end-users if needed

## Debugging Shortcodes

1. **Common Issues**:

   - Shortcode not registered (check `add_shortcode()` call)
   - PHP errors in shortcode function
   - Conflicting shortcode names
   - Improper attribute handling

2. **Debugging Tools**:
   - Enable `WP_DEBUG` in wp-config.php
   - Check for conflicts by disabling other plugins
   - Use `var_dump($atts);` to inspect attributes

## Real-World Example: Creating a Responsive Card Shortcode

```php
<?php
function card_shortcode($atts, $content = null) {
    $atts = shortcode_atts(
        array(
            'title' => '',
            'image' => '',
            'link'  => '#',
            'style' => 'default'
        ),
        $atts,
        'card'
    );
    ob_start(); // Start output buffering
?>

    <div class="card card-<?php echo esc_attr($atts['style']); ?>">
        <?php if($atts['image']): ?>
        <div class="card-image">
            <img src="<?php echo esc_url($atts['image']); ?>" alt="<?php echo esc_attr($atts['title']); ?>">
        </div>
        <?php endif; ?>
        <div class="card-body">
            <h3><?php echo esc_html($atts['title']); ?></h3>
            <div class="card-content">
                <?php echo do_shortcode($content); ?>
            </div>
            <a href="<?php echo esc_url($atts['link']); ?>" class="card-link">Read More</a>
        </div>
    </div>
<?php
    return ob_get_clean(); // Return the buffered output
}
add_shortcode('card', 'card_shortcode');
```

Usage:

```html
[card title="Sample Card" image="http://example.com/image.jpg"
link="/sample-page" style="highlight"] This is the content of my card. It can
include <strong>HTML</strong> and even [other_shortcodes]. [/card]
```
