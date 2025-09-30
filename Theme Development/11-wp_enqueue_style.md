## 📌 `wp_enqueue_style()`

✅ One-liner for interview:

> wp_enqueue_style() is the WP standard to load CSS with handles, dependencies, versions, and media types—keeping styles conflict-free, optimized, and child-theme compatible.

`wp_enqueue_style()` is a function in WordPress that is used to properly load (enqueue) stylesheets in your themes or plugins. It is an essential function for adding CSS files to your WordPress site in a way that avoids conflicts and ensures that styles are loaded in the correct order. Let's dive into its details, purpose, core concepts, and use cases.

### Purpose of `wp_enqueue_style()`

**The main purpose of `wp_enqueue_style()` is to ensure that stylesheets are added to your WordPress site in an organized, reliable way. It handles the dependencies between stylesheets, makes sure that the files are included only once, and helps prevent conflicts with other stylesheets or scripts.**

### Core Concepts of `wp_enqueue_style()`

1. **Hooking the Function**:
   You should always use `wp_enqueue_style()` inside action hooks like `wp_enqueue_scripts` or `admin_enqueue_scripts`. This makes sure that styles are loaded at the appropriate time in the page's lifecycle.

   ```php
   add_action('wp_enqueue_scripts', 'enqueue_custom_styles');

   function enqueue_custom_styles() {
       wp_enqueue_style('my-style', get_template_directory_uri() . '/css/custom-style.css');
   }
   ```

2. **Unique Handle**:
   Each stylesheet that you enqueue must have a unique handle. This handle is used by WordPress to manage the stylesheet and its dependencies.

3. **Dependencies**:
   You can specify dependencies for your stylesheets. This tells WordPress that certain styles need to be loaded before your stylesheet.

4. **Versioning**:
   Version numbers are used to cache-bust your stylesheets when needed. This ensures that browsers always load the latest version of your stylesheet.

5. **Media Queries**:
   You can specify media types (e.g., `screen`, `print`) for the stylesheets to control when they should be applied.

### Basic Syntax of `wp_enqueue_style()`

```php
wp_enqueue_style(
    $handle,           // (string) Required. Unique name for the stylesheet.
    $src,              // (string) Optional. URL to the stylesheet. Default is ''.
    $deps,             // (array) Optional. An array of dependencies. Default is an empty array.
    $ver,              // (string|bool|null) Optional. Version number. Default is false.
    $media             // (string) Optional. The media type for the stylesheet. Default is 'all'.
);
```

### Example: Enqueuing a Basic Stylesheet

Here is a simple example of how to enqueue a custom stylesheet for the front end of your WordPress theme.

```php
function my_theme_enqueue_styles() {
    wp_enqueue_style('main-style', get_stylesheet_uri()); // Adds the main style.css file
    wp_enqueue_style('custom-style', get_template_directory_uri() . '/css/custom-style.css', array('main-style'), '1.0.0', 'all');
}
add_action('wp_enqueue_scripts', 'my_theme_enqueue_styles');
```

In this example:

- `main-style` is the handle for the main stylesheet.
- `custom-style` depends on `main-style` and will be loaded after it.
- The version is set to `1.0.0`, which is useful for cache-busting.
- The `media` parameter is set to `all`, which means it applies to all media types.

### Advanced Use Cases

1. **Enqueuing Google Fonts**
   You can easily add external stylesheets like Google Fonts using `wp_enqueue_style()`.

   ```php
   function enqueue_google_fonts() {
       wp_enqueue_style('google-fonts', 'https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;600&display=swap', false);
   }
   add_action('wp_enqueue_scripts', 'enqueue_google_fonts');
   ```

2. **Conditional Stylesheets**
   Sometimes, you may need to load stylesheets conditionally, such as only for specific pages or posts.

   ```php
   function load_conditional_styles() {
       if (is_page('contact')) {
           wp_enqueue_style('contact-style', get_template_directory_uri() . '/css/contact-style.css');
       }
   }
   add_action('wp_enqueue_scripts', 'load_conditional_styles');
   ```

3. **Admin-Specific Styles**
   To load stylesheets specifically in the WordPress admin dashboard, use the `admin_enqueue_scripts` action.

   ```php
   function admin_custom_styles() {
       wp_enqueue_style('admin-style', get_template_directory_uri() . '/css/admin-style.css');
   }
   add_action('admin_enqueue_scripts', 'admin_custom_styles');
   ```

---

### 🔑 Register vs Enqueue

- **`wp_register_style()` / `wp_register_script()`** → Only _register_ the asset with WordPress (give it a handle, path, dependencies, version, etc.). Nothing is loaded yet.
- **`wp_enqueue_style()` / `wp_enqueue_script()`** → Actually _enqueue_ (load) the asset in the page output.

👉 Think of **registering** as "preparing" and **enqueueing** as "executing".

### 📘 Expanded Example

```php
function aquila_enqueue_scripts() {

    // 1. Register CSS
    wp_register_style(
        'style-css',                           // Handle (unique ID)
        get_stylesheet_uri(),                  // URL (style.css from theme root)
        [],                                    // Dependencies (none)
        filemtime(get_template_directory() . '/style.css'), // Version (cache-busting with file time)
        'all'                                  // Media type (all screens)
    );

    // 2. Register JS
    wp_register_script(
        'main-js',                                            // Handle
        get_template_directory_uri() . '/assets/main.js',     // URL
        ['jquery'],                                           // Dependencies (example: needs jQuery)
        filemtime(get_template_directory() . '/assets/main.js'), // Version
        true                                                  // Load in footer
    );

    // 3. Enqueue (actually load them)
    wp_enqueue_style('style-css');
    wp_enqueue_script('main-js');
}
add_action('wp_enqueue_scripts', 'aquila_enqueue_scripts');
```

### 🧩 Explanation of Each Parameter

#### For **`wp_register_style()`**

```php
wp_register_style( $handle, $src, $deps, $ver, $media );
```

- `$handle` → Unique name (e.g., `style-css`).
- `$src` → URL of stylesheet.
- `$deps` → Dependencies (array of other CSS handles).
- `$ver` → Version (useful for cache-busting, e.g., `filemtime()` returns last modified time).
- `$media` → e.g. `all`, `screen`, `print`.

#### For **`wp_register_script()`**

```php
wp_register_script( $handle, $src, $deps, $ver, $in_footer );
```

- `$handle` → Unique name.
- `$src` → URL of script file.
- `$deps` → Dependencies (e.g., `jquery`, other custom scripts).
- `$ver` → Version (cache-busting).
- `$in_footer` → `true` to load before `</body>`, `false` to load in `<head>`.

#### 🚀 Why Register First?

- **Flexibility** → You can register assets once, then enqueue them only when needed (e.g., only on a contact page).
- **Performance** → Avoids loading unnecessary CSS/JS on all pages.
- **Reusability** → Multiple parts of your theme/plugin can enqueue the same registered asset.

#### 🎯 Conditional Enqueue Example

```php
function aquila_enqueue_scripts() {
    wp_register_script(
        'contact-form-js',
        get_template_directory_uri() . '/assets/contact.js',
        [],
        '1.0.0',
        true
    );

    if (is_page('contact')) {
        wp_enqueue_script('contact-form-js'); // Only load on Contact page
    }
}
add_action('wp_enqueue_scripts', 'aquila_enqueue_scripts');
```

✅ **One-liner for interview**:
"In WordPress, you can `register` styles/scripts with handles, dependencies, and versions, then `enqueue` them only where needed. This keeps loading organized, cache-busted, and optimized."

---

### Best Practices for Using `wp_enqueue_style()`

1. **Use Correct Hooks**: Always use `wp_enqueue_scripts` for front-end styles and `admin_enqueue_scripts` for admin styles.
2. **Avoid Direct Linking**: Avoid directly adding `<link>` tags in your theme's header or footer files. Use `wp_enqueue_style()` to ensure proper handling by WordPress.
3. **Use Version Numbers**: Use versioning to control caching issues. This helps when updating stylesheets to reflect changes immediately.
4. **Respect Dependencies**: Specify dependencies to ensure your stylesheets load in the correct order.
5. **Security**: Use `esc_url()` if you are generating dynamic URLs for your stylesheets to prevent security vulnerabilities.

---

### Common Use Cases in Practice

- **Theme Development**: When creating or customizing themes, you typically use `wp_enqueue_style()` to load your main CSS file and any additional stylesheets you may need.
- **Plugin Development**: Plugins that require styles should use this function to include their CSS files properly without conflicting with other plugins or themes.
- **Performance Optimization**: By managing stylesheets effectively, you can avoid duplicate CSS loading and minimize performance issues caused by unnecessary HTTP requests.

### Example for Theme Customization

Suppose you're developing a child theme and want to enqueue its stylesheet after the parent theme's styles. Here's how to do it:

```php
function child_theme_enqueue_styles() {
    wp_enqueue_style('parent-style', get_template_directory_uri() . '/style.css');
    wp_enqueue_style('child-style', get_stylesheet_directory_uri() . '/style.css', array('parent-style'), wp_get_theme()->get('Version'));
}
add_action('wp_enqueue_scripts', 'child_theme_enqueue_styles');
```

This code ensures that the child theme styles are loaded after the parent theme styles.

### Conclusion

Using `wp_enqueue_style()` is the recommended way to include styles in WordPress because it handles dependencies, order, versioning, and conflicts effectively. By following best practices and understanding how this function works, you ensure that your stylesheets load correctly, keeping your website well-structured and optimized.
