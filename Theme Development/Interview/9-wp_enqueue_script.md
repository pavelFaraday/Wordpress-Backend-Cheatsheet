## 📌 `wp_enqueue_script`

`wp_enqueue_script` is a fundamental function in WordPress used to manage and load JavaScript files efficiently. Understanding its purpose, core concepts, and use cases will help you manage scripts in WordPress better. 

### 1. **What is `wp_enqueue_script`?**

`wp_enqueue_script` is a WordPress function that properly adds JavaScript files to your site. <u>It ensures that scripts are loaded in the correct order, with proper dependencies, and without conflicts. It a structured way to include JavaScript files without directly embedding them into your theme or plugin files.</u>

### 2. **Purpose of Using `wp_enqueue_script`**

The primary purpose of using `wp_enqueue_script` is to avoid issues with JavaScript conflicts and duplicate loading. It allows you to:
- **Ensure Script Dependencies:** Load JavaScript files in a specific order based on their dependencies.
- **Prevent Duplicates:** Avoid the risk of multiple versions of the same script being loaded.
- **Better Performance:** Allow browsers to cache the files and improve page load speed.
- **Global Management:** Manage all scripts centrally in WordPress, improving maintenance.

### 3. **Core Concepts**

Let's go through the main components involved in using `wp_enqueue_script`:

- **Handle:** A unique identifier for your script, which allows WordPress to manage its loading.
- **Source (src):** The URL path to the script file.
- **Dependencies:** An array of handles that specify which other scripts must be loaded before this one.
- **Version:** The version number of your script to handle cache busting.
- **In Footer:** A boolean value that determines whether to load the script in the footer (`true`) or the header (`false`).

### 4. **Basic Syntax of `wp_enqueue_script`**

```php
wp_enqueue_script( $handle, $src, $deps, $ver, $in_footer );
```

- `$handle` *(string)*: A unique name for the script.
- `$src` *(string)*: URL path to the script file.
- `$deps` *(array)*: Optional. An array of handles that this script depends on.
- `$ver` *(string)*: Optional. The script version number.
- `$in_footer` *(bool)*: Optional. Whether to load the script in the footer (`true`) or the header (`false`).

---
---
---

### 5. **Use Cases in Practice**

#### **1. Enqueueing a Custom Script in Your Theme**

Here's a basic example of how to use `wp_enqueue_script` in a theme:

```php
wp_enqueue_script('main-university-js', get_theme_file_uri('/build/index.js'), array('jquery'), '1.0', true);`
```

```php
function my_custom_theme_scripts() {
    wp_enqueue_script(
        'custom-script',                   // Handle
        get_template_directory_uri() . '/js/custom-script.js',  // Source
        array('jquery'),                   // Dependencies
        '1.0.0',                           // Version
        true                               // Load in footer
    );
}
add_action('wp_enqueue_scripts', 'my_custom_theme_scripts');
```

**Explanation**:
- This code enqueues a JavaScript file located in the theme's `js` folder.
- The script depends on jQuery, so it ensures jQuery is loaded first.
- The script will be loaded in the footer for better performance.

#### **2. Using `wp_enqueue_script` in a Plugin**

When you are developing a plugin, it's recommended to use a similar approach:

```php
function my_plugin_enqueue_scripts() {
    wp_enqueue_script(
        'my-plugin-script',
        plugins_url( '/assets/js/my-plugin-script.js', __FILE__ ),
        array('jquery'),
        '1.2.0',
        true
    );
}
add_action('wp_enqueue_scripts', 'my_plugin_enqueue_scripts');
```

This example properly enqueues a script from the plugin's assets folder. It uses `plugins_url()` to get the path to the file, which is more reliable than hardcoding the URL.

### 6. **Handling Script Dependencies**

Dependencies are crucial when you want to ensure that your script loads only after certain scripts are available. For instance:

```php
wp_enqueue_script( 'my-custom-js', get_template_directory_uri() . '/js/custom.js', array('jquery', 'bootstrap-js'), '1.0', true );
```

In this example:
- `custom.js` will only load if both `jquery` and `bootstrap-js` are available.
- This approach ensures that dependencies are met before loading your script, reducing the risk of JavaScript errors.

### 7. **Conditional Script Loading**

Sometimes you might want to load a script only on specific pages or conditions. You can achieve this like so:

```php
function my_conditional_scripts() {
    if ( is_page('contact') ) {
        wp_enqueue_script( 'contact-form-script', get_template_directory_uri() . '/js/contact-form.js', array(), '1.0', true );
    }
}
add_action('wp_enqueue_scripts', 'my_conditional_scripts');
```

This ensures that the `contact-form.js` script is loaded only on the "Contact" page, improving performance by not loading unnecessary scripts.

### 8. **Common Best Practices**

- **Always Use `wp_enqueue_script`:** Avoid directly adding scripts using `script` tags in the header or footer.
- **Load in Footer When Possible:** Loading scripts in the footer (`true`) can improve page speed and user experience.
- **Use Handles and Dependencies Properly:** To prevent conflicts and make sure that dependencies are respected.

### 9. **Debugging Script Issues**

If your scripts are not loading properly, here are a few things to check:
- **Console Errors:** Look at your browser’s developer console for errors.
- **Correct Handle:** Ensure the handle is unique and doesn't conflict with other scripts.
- **Dependencies Loaded:** Verify that all dependencies are loaded before your script.

### 10. **Advanced Use Case: Localizing Scripts**

If you need to pass PHP data to your JavaScript file, use `wp_localize_script()` in combination with `wp_enqueue_script()`:

```php
function my_localized_script() {
    wp_enqueue_script( 'custom-script', get_template_directory_uri() . '/js/custom-script.js', array(), '1.0.0', true );
    
    wp_localize_script( 'custom-script', 'myScriptData', array(
        'ajax_url' => admin_url( 'admin-ajax.php' ),
        'nonce'    => wp_create_nonce( 'my_nonce' )
    ));
}
add_action('wp_enqueue_scripts', 'my_localized_script');
```

This example sends data from PHP to JavaScript, allowing the use of AJAX or other server-side values within your script.

### **Summary**

Using `wp_enqueue_script` properly in WordPress ensures that your JavaScript files are loaded efficiently, in the right order, and without conflicts. Understanding its components, dependencies, and best practices will help you deliver faster, more stable websites.