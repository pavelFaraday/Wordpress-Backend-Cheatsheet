## 📌 `wp_loaded` in WordPress

The `wp_loaded` action in WordPress is a powerful hook that fires once all core files have been loaded, but before the headers are sent and any query is executed. Here's a detailed breakdown of its purpose, core concepts, and practical use cases:

---

### **Purpose of `wp_loaded`**
The `wp_loaded` hook is intended to signal that WordPress is fully loaded and ready to begin processing requests, but no output (like headers or content) has been sent yet. 

This makes it ideal for tasks that:
1. Need to rely on the availability of core WordPress functions, plugins, and themes.
2. Do not interfere with the actual page rendering or request processing.

---

### **Core Concepts of `wp_loaded`**
1. **Execution Timing**:
   - It fires after all WordPress core files, plugins, and theme files are loaded.
   - It is invoked right before WordPress executes its query and sends headers.

2. **Position in the Load Process**:
   - It occurs later than `muplugins_loaded` (used for must-use plugins) and `plugins_loaded` (for general plugins).
   - It is earlier than `init` (where many plugins add their functionality).

3. **Difference from Similar Hooks**:
   - **`plugins_loaded`**: Fires immediately after all plugins are loaded but before the current theme functions file is included.
   - **`init`**: Fired after WordPress has initialized but before any headers are sent.
   - **`wp_loaded`**: Signifies that everything needed to execute WordPress is loaded but no processing has begun.

4. **Global Availability**:
   - At this stage, all global variables like `$wpdb`, `$wp_query`, and `$wp_rewrite` are available for use.

---

### **Practical Use Cases for `wp_loaded`**
1. **Custom Plugin Initialization**:
   Use `wp_loaded` to initialize a plugin that requires WordPress core, plugins, and themes to be fully loaded.
   ```php
   add_action('wp_loaded', 'my_plugin_initialize');
   function my_plugin_initialize() {
       // Code to initialize plugin features
   }
   ```

2. **Registering Custom Scripts or Styles**:
   Register or enqueue scripts/styles at this stage if they depend on conditions or logic that require the core to be loaded.
   ```php
   add_action('wp_loaded', 'register_custom_scripts');
   function register_custom_scripts() {
       if (is_admin()) {
           wp_enqueue_script('custom-admin-script', plugin_dir_url(__FILE__) . 'admin.js', [], '1.0', true);
       }
   }
   ```

3. **Performing Setup Tasks**:
   Perform one-time tasks, such as setting default options or cleaning up transients.
   ```php
   add_action('wp_loaded', 'setup_default_options');
   function setup_default_options() {
       if (!get_option('my_plugin_option')) {
           update_option('my_plugin_option', 'default_value');
       }
   }
   ```

4. **Triggering Custom Events**:
   Use it to trigger custom logic based on conditions, such as user roles or specific settings.
   ```php
   add_action('wp_loaded', 'check_user_role_and_redirect');
   function check_user_role_and_redirect() {
       if (current_user_can('editor') && is_admin()) {
           wp_redirect(home_url());
           exit;
       }
   }
   ```

5. **Dynamic Role or Capability Modifications**:
   Modify user roles or capabilities programmatically.
   ```php
   add_action('wp_loaded', 'modify_roles_and_capabilities');
   function modify_roles_and_capabilities() {
       $role = get_role('editor');
       if ($role && !$role->has_cap('manage_options')) {
           $role->add_cap('manage_options');
       }
   }
   ```

---

### **Example Workflow Using `wp_loaded`**
Here’s how you might use the hook in a plugin:

#### Scenario: A Custom Plugin that Requires Complex Initialization
```php
add_action('wp_loaded', 'custom_plugin_bootstrap');
function custom_plugin_bootstrap() {
    // Check if a required plugin is active
    if (!is_plugin_active('required-plugin/required-plugin.php')) {
        deactivate_plugins(plugin_basename(__FILE__));
        wp_die('This plugin requires the Required Plugin to be active.');
    }

    // Perform conditional setup based on user roles
    if (current_user_can('administrator')) {
        // Log an event or notify the admin
        error_log('Admin logged in');
    }

    // Initialize any custom features
    initialize_custom_features();
}

function initialize_custom_features() {
    // Add custom functionality here
}
```

---

### **Key Considerations**
1. **Avoid Heavy Processing**:
   Since `wp_loaded` runs for every request, keep the logic lightweight to avoid performance bottlenecks.

2. **Not for Query Modifications**:
   If you need to modify the WordPress query, use `pre_get_posts` or similar hooks.

3. **Compatibility**:
   Ensure your code within this hook doesn’t conflict with themes or plugins that rely on similar logic.

---

### **Summary**
- `wp_loaded` is a late-stage hook perfect for tasks that require all WordPress functionality to be ready.
- It’s best suited for initialization tasks, conditional logic, and plugin or theme-specific setup that doesn't affect query execution or page rendering.
- By leveraging `wp_loaded`, you ensure your custom logic integrates smoothly with the core, themes, and plugins without disrupting the request lifecycle.