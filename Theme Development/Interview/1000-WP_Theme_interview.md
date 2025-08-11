# WordPress Theme Development Interview Questions

> **1. Have you created a custom WordPress theme?**

Yes, I have developed multiple custom WordPress themes from scratch. My typical process starts with setting up the theme folder, creating the `style.css` with proper theme headers, and adding the `functions.php` file to handle all scripts and styles using `wp_enqueue_style()` and `wp_enqueue_script()` instead of hardcoding them.

I follow the WordPress template hierarchy to create essential files like `header.php`, `footer.php`, `index.php`, `single.php`, and `archive.php`. For reusable layouts, I use `get_template_part()` to keep the code maintainable.

I also register theme features like `post-thumbnails`, `title-tag`, and `custom-logo` via `add_theme_support()` function. When building dynamic pages, I use the WordPress Loop and, if needed, custom `WP_Query` for advanced filtering.

For front-end styling, I integrate Bootstrap by enqueueing it properly and overriding styles cleanly in a separate custom CSS file to keep it maintainable. I make sure my HTML is semantic, responsive, and accessible, following **WCAG** guidelines.

Overall, my custom themes are lightweight, SEO-friendly, and built to work well with both the block editor and page builders like Elementor when needed.

---

> **2. How would you integrate Bootstrap in a custom theme?**

I integrate Bootstrap into a custom WordPress theme by properly enqueueing its CSS and JavaScript via the `functions.php` file, never by hardcoding it in the header or footer.

For example, I use `wp_enqueue_style()` to load the Bootstrap CSS from a CDN or a local copy, making sure to set a unique handle and correct dependencies. For JavaScript, I enqueue it with `wp_enqueue_script()` and specify jquery as a dependency if needed, setting it to load in the footer for better performance.

I keep my own custom styles in a separate stylesheet that I enqueue <u>after Bootstrap</u>, so my overrides take effect without touching the Bootstrap source files.

**functions.php**

```php
function mytheme_enqueue_bootstrap() {
    // Bootstrap CSS (CDN example)
    wp_enqueue_style(
        'bootstrap-css',
        'https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css',
        array(), // No dependencies
        '5.3.3'
    );

    // Theme's main stylesheet (load after Bootstrap so overrides work)
    wp_enqueue_style(
        'mytheme-style',
        get_stylesheet_uri(),
        array('bootstrap-css'),
        filemtime(get_template_directory() . '/style.css')
    );

    // Bootstrap JS bundle (includes Popper)
    wp_enqueue_script(
        'bootstrap-js',
        'https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js',
        array('jquery'), // jQuery dependency if needed
        '5.3.3',
        true // Load in footer
    );
}
add_action('wp_enqueue_scripts', 'mytheme_enqueue_bootstrap');
```

---

> **3. Can you explain the WordPress Template Hierarchy and why it’s important in theme development?**

The WordPress Template Hierarchy defines the order in which WordPress looks for template files when rendering a page. For example, if I visit a single blog post, WordPress first checks `single-{post_type}.php`, then `single.php`, and finally falls back to `index.php`. This hierarchy is crucial because it allows me to create specific templates for different content types without repeating code unnecessarily. Understanding it ensures I can structure my theme efficiently and override only what’s necessary, keeping it clean and maintainable.

---

> **4. How do you properly enqueue styles and scripts in a WordPress theme?**

I always enqueue styles and scripts via the `wp_enqueue_scripts` action in `functions.php` using `wp_enqueue_style()` and `wp_enqueue_script()`. This method ensures that WordPress handles dependencies, prevents conflicts, and allows plugins to manage or override these assets. For example, I would enqueue Bootstrap first, then my theme’s main CSS, ensuring my overrides load last. For JavaScript, I set the `$in_footer` parameter to `true` to improve performance by loading scripts at the bottom of the page.

---

> **5. How do you make your custom theme compatible with the WordPress Block Editor (Gutenberg)?**

To make my custom theme Gutenberg-compatible, I use `add_theme_support()` in `functions.php` for features like `editor-styles`, `align-wide`, `responsive-embeds`, and custom color palettes. I also enqueue an `editor-style.css` file so that the block editor matches the front-end styles. This ensures that the content authors see a true WYSIWYG experience while editing, and that my theme’s design stays consistent across both the editor and live site.

---

> **6. How would you handle reusable parts like headers, footers, or sidebars in a custom theme?**

For reusable layout elements like headers and footers, I use `get_header()` and `get_footer()` to include them in templates. For smaller components, I use `get_template_part()` so that I can reuse the same piece of markup in multiple templates without duplication. This approach keeps the theme DRY (Don’t Repeat Yourself), makes it easier to update common sections, and improves maintainability.

---

> **7. What steps do you take to ensure your custom theme is responsive and accessible?**

I follow a mobile-first approach in my CSS, using media queries to adjust layouts for larger screens. I also use semantic HTML5 elements like `<header>`, `<main>`, `<nav>`, and `<section>` for better structure. For accessibility, I use ARIA attributes where needed, maintain proper heading hierarchy, and ensure sufficient color contrast. I also test my theme with tools like Lighthouse or Axe to catch any accessibility issues early.

---

> **8. How do you register a Custom Post Type in WordPress, and what are some important arguments you use?**

I register a CPT using the `register_post_type()` function inside the `init` action hook. Some important arguments I set are:

- `public` → Whether it’s visible on the front end.
- `supports` → Which features are available (title, editor, thumbnail, custom fields).
- `rewrite` → For custom permalinks.
- `show_in_rest` → To make it Gutenberg and REST API compatible.
  For example, for a 'Game' CPT, I’d use `supports` to allow title, editor, and thumbnail, set `rewrite` to `slug => 'games'`, and enable `show_in_rest` for block editor compatibility.

---

> **9. What’s the difference between `get_posts()`, `query_posts()`, and `WP_Query()`?**

`get_posts()` is a simple wrapper around `WP_Query` and returns an array of posts without modifying the main query. `query_posts()` replaces the main query — which is not recommended because it can cause pagination issues. `WP_Query()` is the most flexible — it lets you create custom queries without touching the main loop. I prefer `WP_Query()` for advanced filtering, as it gives me full control while keeping the main query intact."

---

> **10. "How would you create a shortcode that accepts attributes and displays dynamic content?"**

I use `add_shortcode()` to register the shortcode and `shortcode_atts()` to set default attribute values. Then I use the attributes to query and display content. For example, `[casino_list type="top"]` could use `$atts['type']` to filter a CPT query and return a list of casinos. I always sanitize attributes using functions like `esc_attr()` or `sanitize_text_field()` to keep the output secure.

---

> **11. How would you display related Custom Post Types, for example, 'Games' related to a 'Casino'?**

There are a few ways:

1. Using a shared taxonomy (e.g., 'Casino Category') and querying posts with `tax_query`.
2. Storing the relationship in post meta and querying with `meta_query`.
3. Using ACF Relationship fields for easier admin management.
   Once I have the related IDs, I run a `WP_Query` to fetch and display them, ensuring I escape all output and use semantic HTML for the markup.

---

> **12. Can you give an example of an advanced WP_Query you’ve written?**

Sure — for example, I once had to display only 'Casino' posts that had a meta field `rating` greater than 4, and also belonged to the 'Europe' taxonomy term. I used `meta_query` with `compare => '>'` for the rating, and `tax_query` with `field => 'slug'` for the taxonomy. I also set `'orderby' => 'meta_value_num'` to sort them by rating in descending order.

```php
<?php
$args = array(
    'post_type'      => 'casino',
    'posts_per_page' => -1, // Or set a specific number
    'meta_query'     => array(
        array(
            'key'     => 'rating',           // Custom field key
            'value'   => 4,                  // Minimum rating
            'type'    => 'NUMERIC',
            'compare' => '>'                 // Greater than 4
        ),
    ),
    'tax_query'      => array(
        array(
            'taxonomy' => 'region',          // Replace with your taxonomy
            'field'    => 'slug',
            'terms'    => 'europe'           // The term slug
        ),
    ),
    'orderby'        => 'meta_value_num',    // Order by numeric meta value
    'meta_key'       => 'rating',            // The meta key to order by
    'order'          => 'DESC'
);

$query = new WP_Query( $args );

if ( $query->have_posts() ) {
    echo '<ul class="casino-list">';
    while ( $query->have_posts() ) {
        $query->the_post();
        echo '<li>' . esc_html( get_the_title() ) . ' - Rating: ' . esc_html( get_post_meta( get_the_ID(), 'rating', true ) ) . '</li>';
    }
    echo '</ul>';
    wp_reset_postdata();
} else {
    echo '<p>No casinos found in Europe with rating above 4.</p>';
}
?>
```

---

> **13. When would you use a Custom Post Type instead of a regular post or page?**

I use a CPT when the content type has a distinct purpose or structure different from blog posts or pages. For example, a 'Game' or 'Casino' post type would have its own template, fields, and archive. CPTs help keep the admin organized, improve UX for content managers, and allow custom queries without mixing unrelated content in the 'Posts' list.

---

> **14. How do you register a taxonomy for a Custom Post Type?14.**

I use `register_taxonomy()` inside the `init` hook, specifying the CPT slug in the second parameter so WordPress associates them. I set arguments like `hierarchical` for category-like behavior, `rewrite` for clean URLs, and `show_in_rest` for block editor compatibility. This way, the taxonomy appears in the admin menu and can be queried on the front end.

---

> **15. How would you create an archive page for a Custom Post Type?**

I create a template file named `archive-{posttype}.php` in my theme folder. WordPress will automatically use it for that CPT’s archive page. Inside, I use The Loop to display posts and can run additional queries if needed. I also ensure the CPT’s `has_archive` argument is set to `true` in `register_post_type()`.

---

> **16. Can you explain how `shortcode_atts()` works in WordPress?**

`shortcode_atts()` lets me define default attribute values for a shortcode and merge them with user-provided values. This ensures that even if the user omits certain attributes, my code has predictable values to work with. For example, if `[gallery type="slider"]` is used without a 'type' attribute, it can default to 'grid'.

---

> **17. How would you pass a Custom Post Type query result into a shortcode output?**

Inside the shortcode callback, I create a `WP_Query` to fetch the CPT data, loop through the results, and build the HTML as a string. I return the HTML instead of echoing it, because shortcodes must return content to be placed in the post/page output.

---

> **18. What’s the difference between `pre_get_posts` and `WP_Query`?**

`WP_Query` creates a completely new query without affecting the main query. `pre_get_posts` modifies the main query before WordPress executes it, making it useful for changing default archive or search behavior globally. I use `pre_get_posts` when I need site-wide behavior changes, and `WP_Query` for targeted, local queries.

---

> **19. How do you create a shortcode that works both in the content area and inside a template file?**

I first register it with `add_shortcode()` so it works in the editor. To use it in a template, I wrap it with `do_shortcode('[my_shortcode attr="value"]')`. This way, the same logic powers both editor content and hardcoded template output.

---

> **20. How can you make a CPT query more efficient when dealing with thousands of posts?**

I use `'no_found_rows' => true` in the query args when pagination isn’t needed, avoid `'posts_per_page' => -1` unless necessary, and cache results with `transients` for repeated queries. I also only request the fields I need using `'fields' => 'ids'` if I just need post IDs.

---

> **21. How do you ensure a WP_Query result is reset after a custom loop?**

I always call `wp_reset_postdata()` after using `WP_Query` so that global variables like `$post` are restored to the main query’s state. This prevents unexpected behavior in subsequent template code.

---

> **22. Can a shortcode output be affected by filters? How would you implement that?**

Yes — I can wrap my shortcode output in a filter, for example:

```php
return apply_filters('my_shortcode_output', $output);
```

This allows other developers or plugins to modify the output without editing the shortcode code directly, improving extensibility.

---

> **23. When to use Custom Field registration with code instead of ACF?**

I prefer registering custom fields with code when I want full control over performance, maintainability, and portability.

ACF is great for quick, non-technical setups, but it adds a layer of abstraction and extra database queries.

If the field is core to theme or plugin functionality — for example, a "featured image credit" field for every post type — I’ll register it in functions.php or a plugin using register_meta() or add_meta_box().

**This ensures:**

- No dependency on a plugin (safer for distribution or client handover)
- Better version control (fields live in the code repository)
- More predictable performance (no extra UI overhead)

In short, I use code for essential, permanent fields and ACF for dynamic, non-critical, client-managed fields.

---

> **24. When to use CPT registration with code instead of CPT UI plugin?**

I use `register_post_type()` in code instead of CPT UI when the post type is part of the site’s permanent architecture or when I need precise control over labels, capabilities, or REST API integration.

CPT UI is great for quick setups during prototyping, but it stores CPT settings in the database, which makes it harder to track in version control or replicate across environments.

**Code-based registration:**

- Keeps CPTs in the theme/plugin repo (easier to maintain across dev/staging/prod)

- Allows custom arguments (capabilities, show_in_rest, custom rewrite rules)

- Ensures CPTs remain functional even if a plugin is disabled

I typically use CPT UI only for temporary or experimental post types during the design phase, then move them to code for production.

---
