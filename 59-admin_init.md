## 📌 `admin_init` Action in WordPress

The `admin_init` action is a core WordPress hook that fires every time an admin page is initialized. This action is a fundamental part of WordPress's admin-side functionality and provides developers with a reliable entry point to execute custom code on any admin screen.

---

### **Purpose of `admin_init`**

The `admin_init` hook is used for tasks that need to run during the initialization of the WordPress admin area. It allows developers to:

1. **Initialize Settings**:
   - Register settings using the `register_setting` function.
2. **Check User Permissions**:
   - Ensure only authorized users can access certain functionality.
3. **Load Resources**:
   - Enqueue admin-specific scripts and styles.
4. **Perform Redirects**:
   - Redirect users based on custom logic.
5. **Handle Form Submissions**:
   - Process custom form submissions sent from admin pages.

---

### **Core Concepts**

#### **When It Runs**
- The `admin_init` hook runs after WordPress loads all the required files for an admin page but before the page content is generated. 
- This ensures that settings, options, or any initialization code is executed before rendering the admin interface.

#### **Execution Context**
- It is specific to the admin area (`wp-admin`) and does not fire on front-end pages.
- Accessible on every admin page, making it versatile for global admin tasks.

#### **Priority and Arguments**
- Like other WordPress hooks, `admin_init` accepts a priority parameter. By default, it runs at priority 10.
- It does not pass any arguments to the callback function.

---

### **Use Cases in Practice**

#### **1. Registering Settings**
The most common use case is to register custom settings for use in the WordPress admin interface.

```php
add_action('admin_init', 'register_custom_settings');
function register_custom_settings() {
    register_setting('my_custom_group', 'my_custom_option');
}
```

#### **2. Redirecting Users**
You can redirect users to a different page based on custom logic.

```php
add_action('admin_init', 'redirect_non_admin_users');
function redirect_non_admin_users() {
    if (!current_user_can('manage_options') && is_admin()) {
        wp_redirect(home_url());
        exit;
    }
}
```

#### **3. Enqueuing Admin Scripts and Styles**
Load admin-specific assets.

```php
add_action('admin_init', 'enqueue_admin_styles');
function enqueue_admin_styles() {
    wp_enqueue_style('custom-admin-style', get_template_directory_uri() . '/admin-style.css');
}
```

#### **4. Processing Form Submissions**
Handle form submissions sent via `POST`.

```php
add_action('admin_init', 'process_custom_form');
function process_custom_form() {
    if (isset($_POST['custom_form_submit']) && check_admin_referer('custom_nonce_action', 'custom_nonce_field')) {
        // Process form data
        update_option('custom_option', sanitize_text_field($_POST['custom_input']));
        wp_redirect(admin_url('options-general.php?page=custom-page&updated=true'));
        exit;
    }
}
```

#### **5. Custom Admin Notices**
Set up admin notices dynamically.

```php
add_action('admin_init', 'set_custom_admin_notice');
function set_custom_admin_notice() {
    if (!get_option('custom_notice_dismissed')) {
        add_action('admin_notices', function () {
            echo '<div class="notice notice-warning is-dismissible">
                <p>This is a custom admin notice!</p>
            </div>';
        });
    }
}
```

#### **6. Restricting Page Access**
Prevent access to specific admin pages.

```php
add_action('admin_init', 'restrict_admin_page_access');
function restrict_admin_page_access() {
    if (is_admin() && isset($_GET['page']) && $_GET['page'] === 'restricted-page') {
        if (!current_user_can('edit_posts')) {
            wp_die('You do not have permission to access this page.');
        }
    }
}
```

#### 7. **Redirect subscriber accounts out of admin and onto homepage**
```php
function redirectSubsToFrontend() {
  $ourCurrentUser = wp_get_current_user();
  if (count($ourCurrentUser->roles) == 1 AND $ourCurrentUser->roles[0] == 'subscriber') {
    wp_redirect(site_url('/'));
    exit;
  }
}
add_action('admin_init', 'redirectSubsToFrontend');

```
---

### **Key Points to Remember**
1. **Global Availability**: It runs on all admin pages, so ensure your code is scoped correctly to avoid unintended behavior.
2. **Proper Checks**: Always verify user permissions and nonce fields when processing sensitive data.
3. **Performance**: Avoid heavy processing inside the `admin_init` action to maintain admin interface performance.

By leveraging the `admin_init` hook effectively, you can build powerful, dynamic, and secure admin-side functionality in your WordPress projects.